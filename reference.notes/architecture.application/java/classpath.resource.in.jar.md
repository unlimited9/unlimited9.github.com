## Spring Boot ClassPathResource in a JAR

#### error.message
```
Caused by: java.nio.file.FileSystemNotFoundException: null
        at jdk.zipfs/jdk.nio.zipfs.ZipFileSystemProvider.getFileSystem(ZipFileSystemProvider.java:169)
        at jdk.zipfs/jdk.nio.zipfs.ZipFileSystemProvider.getPath(ZipFileSystemProvider.java:155)
        at java.base/java.nio.file.Path.of(Path.java:208)
        at java.base/java.nio.file.Paths.get(Paths.java:98)
...
```


#### org.apache.commons:commons-configuration2
add org.apache.commons.configuration2.io.ClasspathLocationStrategy to org.apache.commons.configuration2.io.FileLocationStrategy
```
  //org.apache.commons.configuration2
  private Configuration getConfiguration(String path) {
    Parameters params = new Parameters();

    List<FileLocationStrategy> subs = Arrays.asList(new ClasspathLocationStrategy(), new FileSystemLocationStrategy()); //new ProvidedURLLocationStrategy(),
    FileLocationStrategy strategy = new CombinedLocationStrategy(subs);

    FileBasedConfigurationBuilder<FileBasedConfiguration> builder = new FileBasedConfigurationBuilder<FileBasedConfiguration>(PropertiesConfiguration.class).configure(
            params.properties().setFileName(path).setLocationStrategy(strategy).setListDelimiterHandler(new DefaultListDelimiterHandler(','))
    );

    try {
      return builder.getConfiguration();
    } catch (ConfigurationException e) {
      e.printStackTrace();
    }

    return null;
  }
```
