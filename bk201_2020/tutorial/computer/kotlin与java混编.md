1. idea tools->kotlin->configure kotlin in project
2. kotlin-maven-plugin 插件需要放maven-compiler-plugin之前
3. kotlin-maven-plugin 修改
```xml
                <executions>
                    <execution>
                        <id>compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
-- 选择性加上
                        <configuration>
                            <sourceDirs>
                                <source>src/main/java</source>
                                <source>src/main/kotlin</source>
                            </sourceDirs>
                        </configuration>
-- 选择性加上
                    </execution>
                </executions>
```
4. maven-compiler-plugin修改（重点）

```xml
<executions>
                    <execution>
                        <id>default-compile</id>
                        <phase>none</phase>
                    </execution>
                    <execution>
                        <id>default-testCompile</id>
                        <phase>none</phase>
                    </execution>
                    <execution>
                        <id>compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>testCompile</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>

```



