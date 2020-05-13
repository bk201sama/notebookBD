1. idea tools->kotlin->configure kotlin in project
2. kotlin-maven-plugin 插件需要放maven-compiler-plugin之前
3.kotlin-maven-plugin
```xml
                <executions>
                    <execution>
                        <id>compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <sourceDirs>
                                <source>src/main/java</source>
                            </sourceDirs>
                        </configuration>
                    </execution>
                </executions>
```


