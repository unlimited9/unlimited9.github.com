
## springboot : tomcat
#### Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986
```
@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
    TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
    factory.addConnectorCustomizers(new TomcatConnectorCustomizer() {
        @Override
        public void customize(Connector connector) {
            connector.setProperty("relaxedQueryChars", "|{}[]");
        }
    });
    return factory;
}
```
