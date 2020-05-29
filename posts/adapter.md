### 类适配器模式

`Adapter`类继承`src`类, 实现`dst`接口, 完成`src -> dst`的适配

```java
public class App {

    public static void main(String[] args) {
        new Phone().charge(new VoltageAdapter());
    }
}

class Phone {
    public void charge(Voltage5 voltage5) {
        if (voltage5.output5V() == 5) {
            System.out.println("电压为5V, 能够充电");
        } else {
            System.out.println("电压错误, 不能充电");
        }
    }
}

class Voltage220 {
    public int output220V() {
        return 220;
    }
}

interface Voltage5 {
    int output5V();
}

class VoltageAdapter extends Voltage220 implements Voltage5 {

    @Override
    public int output5V() {
        final int output220V = output220V();
        // 转换过程
        return output220V / 44;
    }
}
```

### 对象适配器模式

`Adapter`不继承`src`类, 而是持有`src`类的对象实例, 以解决强耦合关系。即: 持有`src`类, 实现`dst`接口, 完成`src -> dst`的转换

```java
public class App {

    public static void main(String[] args) {
        final Voltage220 voltage220 = new Voltage220();
        final VoltageAdapter adapter = new VoltageAdapter(voltage220);
        new Phone().charge(adapter);
    }
}

class Phone {
    public void charge(Voltage5 voltage5) {
        if (voltage5.output5V() == 5) {
            System.out.println("电压为5V, 能够充电");
        } else {
            System.out.println("电压错误, 不能充电");
        }
    }
}

class Voltage220 {
    public int output220V() {
        return 220;
    }
}

interface Voltage5 {
    int output5V();
}

class VoltageAdapter implements Voltage5 {

    private Voltage220 voltage220;

    public VoltageAdapter(Voltage220 voltage220) {
        this.voltage220 = voltage220;
    }

    @Override
    public int output5V() {
        final int output220V = voltage220.output220V();
        // 转换过程
        return output220V / 44;
    }
}
```

### 接口适配器模式

提供接口空实现, 以解决不想实现全部方法的情景

```java
public class App {

    public static void main(String[] args) {
        new InterfaceAdapter() {
            @Override
            public void do1() {
                System.out.println("do1");
            }
        }.do1();
    }
}

interface Interface1 {
    void do1();

    void do2();

    void do3();

    void do4();

    void do5();
}

abstract class InterfaceAdapter implements Interface1 {
    @Override
    public void do1() {

    }

    @Override
    public void do2() {

    }

    @Override
    public void do3() {

    }

    @Override
    public void do4() {

    }

    @Override
    public void do5() {

    }
}
```

### Spring中的应用

org.springframework.web.servlet.DispatcherServlet#doDispatch

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

        // 位置1
				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}
				
        // 位置2
				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```

