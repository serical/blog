```java
package com.mutou;

import cn.hutool.core.util.ReUtil;
import com.google.common.collect.Lists;
import lombok.extern.slf4j.Slf4j;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.io.IOException;
import java.util.List;
import java.util.Optional;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Collectors;

@Slf4j
public class JsoupDemo {

    private final static AtomicInteger count = new AtomicInteger();

    public static void main(String[] args) throws IOException {
        String url = "https://www.cyone.com.cn/Article/ruhechuangye/";
        final Document document = Jsoup.connect(url).get();

        final List<String> categoryList = getCategoryList(document);
        for (String categoryUrl : categoryList) {
            final List<String> pageList = getPageList(categoryUrl);
            for (String pageUrl : pageList) {
                final List<String> articleList = getArticleList(pageUrl);
                for (String articleUrl : articleList) {
                    getArticleContent(articleUrl);
                }
            }
        }
    }

    private static List<String> getCategoryList(Document document) {
        final Elements select = document.select("body > div.content > div > div.left > div.main_content > div.box4_m > a");
        return select.stream().map(v -> v.attr("abs:href")).collect(Collectors.toList());
    }

    private static List<String> getPageList(String url) throws IOException {
        String template = "index_%s.html";
        final Document document = Jsoup.connect(url).get();
        final Elements select = document.select("#showpage > div > a:contains(尾页)");
        final String href = select.first().attr("href");
        final int page = Integer.parseInt(ReUtil.getGroup1("index_(.+).html", href));

        List<String> urlList = Lists.newArrayList();
        urlList.add(url);
        for (int i = 2; i <= page; i++) {
            urlList.add(url + String.format(template, page));
        }
        return urlList;
    }

    private static List<String> getArticleList(String url) throws IOException {
        final Document document = Jsoup.connect(url).get();
        final Elements select = document.select("body > div.content > div > div.left > div.main_content > div > div > div > a");
        return select.stream().map(v -> v.attr("abs:href")).collect(Collectors.toList());
    }

    private static void getArticleContent(String url) throws IOException {
        final Document document = Jsoup.connect(url).get();
        final String title = getContent(document, "body > div.content > div.main > div > div.main_content > h1", "提取标题错误");
        final String subTitle = getContent(document, "body > div.content > div.main > div > div.main_content > div.FIELDSET", "提取导读错误");
        final String content = getContent(document, "body > div.content > div.main > div > div.main_content > div.left_co", "提取正文错误");

        log.info("\n 第{}文章 \n title: {}\n subTitle:{}\n content:{}\n\n\n", count.incrementAndGet(), title, subTitle, content);
    }

    private static String getContent(Document document, String selector, String defaultValue) {
        return Optional.ofNullable(document.select(selector))
                .filter(v -> v.size() > 0)
                .map(v -> v.get(0))
                .map(Element::html)
                .orElse(defaultValue);
    }
}
```

