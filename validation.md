# Validation  
## 自訂錯誤訊息  
通常會用@Size, @NotNull, @NotBlank等來達到錯誤檢查，不用寫太多程式碼即可讓Java or Spring來處理，不過，通常都會有預設的錯誤訊息；如果同一個註解用在不同的欄位時，想要有不同的錯誤提示訊息，以下提供一種方式來完成此目的，我想應該還有其他不同的方式也可以做到，等到我打通任督二脈後，我也可以只看官方source code就知道怎麼做，不用一直google，期待那天快到來。  

做法是把註解的訊息給定一個類似變數名稱，讓程式啓動時載入某個訊息檔，之後發生錯誤就去那個訊息檔查找訊息。  

![流程概念圖](https://i.imgur.com/F5nTh9V.png)
流程概念圖  

訊息檔：/src/main/resources/messages.properties  
```shell=1
email.notempty=Please provide valid email id.
```

某一個註解  
```java=1
@NotNull(message = "{email.notempty}")
private RequestBodyObj requestBody;
```

在Controller layer寫兩個Bean，讓Spring auto scan  
```java=1
@Bean
public MessageSource messageSource() {
    ReloadableResourceBundleMessageSource messageSource
      = new ReloadableResourceBundleMessageSource();

    messageSource.setBasename("classpath:messages");
    messageSource.setDefaultEncoding("UTF-8");
    return messageSource;
}

@Bean
public LocalValidatorFactoryBean getValidator() {
    LocalValidatorFactoryBean bean = new LocalValidatorFactoryBean();
    bean.setValidationMessageSource(messageSource());
    return bean;
}
```
