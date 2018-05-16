1.自定义异常处理
@ControllerAdvice + @ExceptionHandler 全局处理 Controller 层异常

例子：
@ControllerAdvice
@ResponseBody
public class GlobalExceptionAdvice {
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MissingServletRequestParameterException.class)
    public Res handleMissingServletRequestParameterException(MissingServletRequestParameterException e) {
        logger.error("缺少请求参数", e);
        return Res.error("required_parameter_is_not_present");
    }
    /**
     * 业务逻辑BusinessException
     * 500 - Internal Server Error
     */
    @ResponseStatus(HttpStatus.OK)
    @ExceptionHandler(BusinessException.class)
    public Res handleServiceException(BusinessException e) {
        logger.error("业务逻辑异常: ", e);
        return Res.custom(500, e.getMessage(), null);
    }
}

/**
 * 自定义异常类
 * Created by 003914[panlc] on 2017-06-08.
 */
public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}

/**
 *自定义异常工具
 */
public class ExceptionUtil {

    /**
     *根据异常提示，返回异常
     * @param key
     * @return
     */
    public static BusinessException getException(String key){
        String message = CgaStringUtil.getProperties(key);
        BusinessException businessException = new BusinessException(message);
        return businessException;
    }

}
/**
*业务层抛出异常示例
*/
public int saveBaseItemInfo(Map<String, Object> map) {
        //必填项筛选
        if (map.get("item_num") == null || StringUtils.isBlank(map.get("item_num").toString())) {
            throw ExceptionUtil.getException("material.item.hasEmpty");   //请求参数错误
        }
}
