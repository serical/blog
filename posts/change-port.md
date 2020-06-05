### API修改某工具端口

```go
package main

import (
	"bytes"
	"encoding/json"
	"github.com/gin-gonic/gin"
	"io/ioutil"
	"math/rand"
	"net/http"
	"os/exec"
	"text/template"
	"time"
)

//const configPath = "/Users/serical/Downloads/config.json"
//const restartCommand = "ls /Users/serical/Downloads"

const configPath = "/etc/v2ray/config.json"
const restartCommand = "systemctl restart v2ray"

const configTemplate = `
{
  "inbounds": [{
    "port": {{.}},
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "你的UUID",
          "level": 1,
          "alterId": 64
        }
      ]
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}


`

func main() {
	r := gin.Default()

	auth := r.Group("/api", gin.BasicAuth(gin.Accounts{
		"用户名": "登录密码",
	}))

	auth.GET("/refresh", func(c *gin.Context) {
		result := refresh()
		c.JSON(http.StatusOK, gin.H{"status": 0, "message": result})
	})

	auth.GET("/port", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"status": 0, "port": readPort()})
	})

	_ = r.Run(":8080")
}

func refresh() string {
	// 读取模板
	t := template.Must(template.New("config").Parse(configTemplate))

	// 随机端口
	rand.Seed(time.Now().UnixNano())
	maxPort := int32(65000)
	minPort := int32(4000)
	port := minPort + rand.Int31n(maxPort-minPort)

	// 模板+变量-->配置文件
	var buffer bytes.Buffer
	_ = t.Execute(&buffer, port)

	// 写入配置
	_ = ioutil.WriteFile(configPath, buffer.Bytes(), 0644)

	// 重启服务
	command := exec.Command("sh", "-c", restartCommand)
	var out bytes.Buffer
	command.Stdout = &out
	_ = command.Run()
	return out.String()
}

func readPort() int32 {
	// 读取配置文件
	file, _ := ioutil.ReadFile(configPath)

	// 解析配置获取端口
	var result map[string]interface{}
	_ = json.Unmarshal(file, &result)
	port := result["inbounds"].([]interface{})[0].(map[string]interface{})["port"].(float64)
	return int32(port)
}

```

