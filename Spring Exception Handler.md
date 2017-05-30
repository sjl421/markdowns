# Spring Exception Handler

先看史进的代码：

```java
@ControllerAdvice
public class ExceptionPorcesser {
    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<ErrorResponse> exceptionHandler(RuntimeException e) {
        ErrorResponse errorResponse = new ErrorResponse();
        errorResponse.errorCode = 1;
        errorResponse.errorMessage = e.getMessage();

        // TODO, delete later
        e.printStackTrace();

        return new ResponseEntity<ErrorResponse>(errorResponse, HttpStatus.EXPECTATION_FAILED);
    }
}
```

史进的代码里面里面通过`@ExceptionHandler`注解让`exceptionHandler()`方法成为了异常处理方法，并且限定了了该方法只能处理`RuntimeException`的异常，

同时又通过使用`@ControllerAdvice`注解来让这个类`ExceptionPorcesser`中的异常处理方法所用于整个web应用，也就是说这个类中的异常处理方法将会处理将处理所有Controller抛出的异常（除非某种异常处理在`@ControllerAdvice`没有声明）；特别强调的是`Controller抛出的异常`:

1. 如果你的Controller底层调用的方法如果产生了异常，如果该异常在Controller层以及Controller层以下都已经被处理了，那么这个异常不会被@ControllerAdvice的类来处理；
2. Controller 主线程起来的子线程抛出的异常也不会被@ControllerAdvice类处理；
3. 只有从主线程抛出的并且到Controller这一层还没有被捕获处理的异常才由ControllerAdvice处理；