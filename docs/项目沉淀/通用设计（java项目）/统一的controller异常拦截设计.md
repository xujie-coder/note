我们的项目中基于@ControllerAdvice 实现了统一异常处理，针对 Controller层的异常做了统一的包装和处理。如以下，是我们定义的全局异常处理器：

```plain
import com.google.common.collect.Maps;
import exception.BizException;
import exception.SystemException;
import org.springframework.http.HttpStatus;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;
import vo.Result;

import java.util.HashMap;
import java.util.Map;

/**
 * 统一的controller异常拦截
 *
 * @author xujie
 * @since 2025/01/09 15:44
 */
@ControllerAdvice
public class GlobalWebExceptionHandler {

    /**
     * 自定义方法参数校验器异常处理器
     *
     * @param exception 方法参数验证失败异常
     * @return
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ResponseBody
    public Map<String,String> handleValidationExceptions(MethodArgumentNotValidException exception){
        HashMap<String,String> errors = Maps.newHashMapWithExpectedSize(1);
        exception.getBindingResult().getAllErrors().forEach((error) ->{
            String name = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(name,errorMessage);
        });
        return errors;
    }

    /**
     * 自定义业务异常处理器
     *
     * @param bizException
     * @return
     */
    @ExceptionHandler(BizException.class)
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Result bizExceptionHandler(BizException bizException){
        Result result = new Result<>();
        result.setCode(bizException.getErrorCode().getCode());
        result.setMessage(bizException.getErrorCode().getMessage());
        result.setSuccess(false);
        return result;
    }

    /**
     * 自定义系统异常处理器
     *
     * @param systemException
     * @return
     */
    @ExceptionHandler(SystemException.class)
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Result systemExceptionHandler(SystemException systemException){
        Result result = new Result<>();
        result.setCode(systemException.getErrorCode().getCode());
        result.setMessage(systemException.getErrorCode().getMessage());
        result.setSuccess(false);
        return result;
    }
}
```

