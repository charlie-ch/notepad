### 1.springboot i18n 国际化
**java**
```
//地区
Locale locale = org.springframework.context.i18n.LocaleContextHolder.getLocale();
//根据地区获得相应字符串  code为key,args可控，defaultMessage可空
org.springframework.context.MessageSource.getMessage(String code,Object[]args,String defaultMessage,Locale locale)
```
**application.properties**
```
### i18n setting.
#
spring.messages.basename=i18n/messages
# Set whether to always apply the MessageFormat rules, parsing even messages without arguments. 
spring.messages.always-use-message-format=false
# Loaded resource bundle files cache expiration, in seconds. When set to -1, bundles are cached forever. 
spring.messages.cache-seconds=-1
# Message bundles encoding. 
spring.messages.encoding=UTF-8
# Set whether to fall back to the system Locale if no files for a specific Locale have been found.
spring.messages.fallback-to-system-locale=true
```
**各语言配置文件**
```
--resource
  --i18n
    --messages_en_US.properties
    --messages_zh_CN.properties
    --messages.properties
 ```
### 2.String format
```
String s = String.format("%tf",new Date());//yyyy-MM-dd格式的
```
