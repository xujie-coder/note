实际就是通过AOP实现拦截，思路就是

1. 定义一个注解
2. 通过AOP去发现使用过该注解的类，对该类的方法进行代理处理，增加额外的逻辑，譬如参数校验、缓存、日志打印等。

## 首先定义一个注解`@Facade`
```plain
/**
 * 自定义注解：统一RPC包装
 *
 * @author xujie
 * @since 2025/01/08 16:51
 */
public @interface Facade {
}
```

## 切面类
```plain
import com.alibaba.fastjson2.JSON;
import exception.BizException;
import exception.SystemException;
import jakarta.validation.ValidationException;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.time.StopWatch;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import response.BaseResponse;
import response.ResponseCode;
import utils.BeanValidator;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Arrays;

/**
 * Facade的切面处理类，统一进行参数校验及异常捕获
 *
 * @author xujie
 * @since 2025/01/08 16:53
 */
@Component
@Aspect
@Slf4j
public class FacadeAspect {

    @Around("@annotation(rpc.Facade)")
    public Object facade(ProceedingJoinPoint pjp) throws Exception {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        Method method = ((MethodSignature) pjp.getSignature()).getMethod();
        Class returnType = ((MethodSignature) pjp.getSignature()).getMethod().getReturnType();
        Object[] args = pjp.getArgs();
        log.info("start to execute, method = {}, args = {}",JSON.toJSONString(method.getName()),JSON.toJSONString(args));

        for (Object parameter : args) {
            try {
                BeanValidator.validateObject(parameter);
            } catch (ValidationException e) {
                printLog(stopWatch,method,args,"failed to validate",null,e);
                return getFailedResponse(returnType,e);

            }
        }

        try {
            Object response = pjp.proceed();
            enrichObject(response);
            printLog(stopWatch,method,args,"failed to validate",response,null);
            return response;
        } catch (Throwable e) {
            printLog(stopWatch,method,args,"failed to validate",null,e);
            return getFailedResponse(returnType,e);
        }

    }


    /**
     * 日志打印
     *
     * @param stopWatch
     * @param method
     * @param args
     * @param action
     * @param response
     * @param throwable
     */
    private void printLog(StopWatch stopWatch, Method method, Object[] args, String action, Object response,
                          Throwable throwable){
        try {
            log.info(getInfoMessage(action,stopWatch,method,args,response,throwable));
        } catch (Exception e) {
            log.error("log failed",e);
        }
    }


    /**
     * 统一格式输出，方便做日志统计
     *
     * *** 如果此处调整日志格式，需要同步调整日志监控 ***
     *
     * @param action 行为
     * @param stopWatch 耗时
     * @param method 方法
     * @param args 参数
     * @param response 响应
     * @param exception 异常
     * @return 拼接后的字符串
     */
    private String getInfoMessage(String action,StopWatch stopWatch,Method method,Object[] args,Object response,
                                  Throwable exception){
        StringBuilder stringBuilder = new StringBuilder(action);
        stringBuilder.append(" ,method = ");
        stringBuilder.append(method.getName());
        stringBuilder.append(" ,cost = ");
        stringBuilder.append(stopWatch.getTime()).append(" ms");
        if (response instanceof BaseResponse) {
            stringBuilder.append(" ,success = ");
            stringBuilder.append(((BaseResponse) response).getSuccess());
        }

        if (exception != null) {
            stringBuilder.append(" ,success = ");
            stringBuilder.append(false);
        }
        stringBuilder.append(" ,args = ");
        stringBuilder.append(JSON.toJSONString(Arrays.toString(args)));

        if (response != null) {
            stringBuilder.append(" ,resp = ");
            stringBuilder.append(JSON.toJSONString(response));
        }

        if (exception != null) {
            stringBuilder.append(" ,exception = ");
            stringBuilder.append(exception.getMessage());
        }

        if (response instanceof BaseResponse) {
            if (!((BaseResponse) response).getSuccess()) {
                stringBuilder.append(" , execute_failed");
            }
        }

        return stringBuilder.toString();
    }

    /**
     * 将response的信息补全，主要是code
     *
     * @param response
     */
    private void enrichObject(Object response){
        if (response instanceof BaseResponse) {
            if (((BaseResponse) response).getSuccess()) {
                // 如果状态是成功的，需要将未设置的responseCode设置成SUCCESS
                if (StringUtils.isEmpty(((BaseResponse) response).getResponseCode())) {
                    ((BaseResponse) response).setResponseCode(ResponseCode.SUCCESS.name());
                }
            }
            else {
                // 如果状态是失败的，需要将未设置的responseCode设置成BIZ_ERROR
                if (StringUtils.isEmpty(((BaseResponse) response).getResponseCode())) {
                    ((BaseResponse) response).setResponseCode(ResponseCode.BIZ_ERROR.name());
                }
            }
        }
    }

    /**
     * 定义并返回一个通用失败响应
     *
     * @param returnType 返回类型
     * @param throwable 异常
     * @return 通用失败响应
     */
    private Object getFailedResponse(Class returnType, Throwable throwable)
            throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        if (returnType.getDeclaredConstructor().newInstance() instanceof BaseResponse) {
            BaseResponse response = (BaseResponse) returnType.getDeclaredConstructor().newInstance();
            response.setSuccess(false);
            if (throwable instanceof BizException bizException) {
                response.setResponseMessage(bizException.getMessage());
                response.setResponseCode(bizException.getErrorCode().getCode());
            } else if (throwable instanceof SystemException systemException) {
                response.setResponseMessage(systemException.getMessage());
                response.setResponseCode(systemException.getErrorCode().getCode());
            }else {
                response.setResponseMessage(throwable.toString());
                response.setResponseCode(ResponseCode.BIZ_ERROR.name());
            }

            return response;
        }
        log.error("failed to getFailedResponse ,returnType (" + returnType + ") is not instanceof BaseResponse");
        return null;
    }

}
```

