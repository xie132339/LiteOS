# CyCloneDX格式 
## 1. 生成 SBOM文件
```mvn 执行生成SBOM文件
# 直接生成sbom
mvn cyclonedx:makeBom

# 依赖配置文件
mvn clean package
```
生成后的文件在/target下 bom.json、bom.xml
* java CyCloneDX Maven插件 https://gitcode.com/gh_mirrors/cy/cyclonedx-maven-plugin
* python CyCloneDX-Python https://github.com/CycloneDX/cyclonedx-python
* 其他 cdxgen https://github.com/CycloneDX/cdxgen
```js 依赖
<plugin>
    <groupId>org.cyclonedx</groupId>
    <artifactId>cyclonedx-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>make-bom</id>
            <phase>package</phase> <!-- 在package阶段生成SBOM -->
            <goals>
                <goal>makeBom</goal>
            </goals>
        </execution>
    </executions>
  <configuration>
    <!-- 基本配置 -->
    <outputFormat>xml</outputFormat> <!-- 输出格式：xml | json | all -->
    <outputName>bom</outputName> <!-- 输出文件名（默认bom） -->
    <outputDirectory>${project.build.directory}</outputDirectory> <!-- 输出目录 -->

    <!-- 包含/排除配置 -->
    <includeCompileScope>true</includeCompileScope> <!-- 包含compile范围依赖 -->
    <includeRuntimeScope>true</includeRuntimeScope> <!-- 包含runtime范围依赖 -->
    <includeProvidedScope>true</includeProvidedScope> <!-- 包含provided范围依赖 -->
    <includeTestScope>false</includeTestScope> <!-- 包含test范围依赖 -->
    <includeSystemScope>true</includeSystemScope> <!-- 包含system范围依赖 -->

    <!-- 组件过滤 -->
    <excludeGroupIds>com.example.internal</excludeGroupIds> <!-- 排除特定groupId的组件 -->
    <excludeArtifactIds>test-utils</excludeArtifactIds> <!-- 排除特定artifactId的组件 -->
    <excludeClassifier>sources,javadoc</excludeClassifier> <!-- 排除特定分类器的组件 -->

    <!-- 高级配置 -->
    <schemaVersion>1.5</schemaVersion> <!-- CycloneDX schema版本 -->
    <includeBomSerialNumber>true</includeBomSerialNumber> <!-- 包含BOM序列号 -->
    <includeLicenseText>false</includeLicenseText> <!-- 包含完整许可证文本 -->
    <resolveLicenses>true</resolveLicenses> <!-- 尝试解析组件许可证 -->
    <failOnError>true</failOnError> <!-- 生成失败时构建失败 -->
    <verbose>false</verbose> <!-- 启用详细日志 -->

    <!-- 特殊配置 -->
    <includeProject=false</includeProject> <!-- 包含项目自身作为组件 -->
    <includeProcessInfo>false</includeProcessInfo> <!-- 包含构建过程信息 -->
    <includeProperties>true</includeProperties> <!-- 包含组件属性 -->
  </configuration>
</plugin>
```
## 2. 使用 dependency-check 生成并查看报告
```mvn 
mvn dependency-check:check
```
```js 依赖包
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>8.4.0</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <format>ALL</format> <!-- 生成多种格式报告 -->
        <failBuildOnCVSS>7</failBuildOnCVSS> <!-- CVSS评分≥7时构建失败 -->
        <suppressionFiles>
            <suppressionFile>dependency-suppression.xml</suppressionFile>
        </suppressionFiles>
    </configuration>
</plugin>
```
# Dependency-Track
* maven 依赖 https://github.com/pmckeown/dependency-track-maven-plugin
## 配置
```js
Dependency-Track apiKey 获取 UI ->  Administration -> Teams -> Create Teams
DEPENDENCY_TRACK_API_KEY = odt_EHu8UP1a_3WvE8y3ftfUWd8OkfgkZ8TAieVt6clnF
```
## 上传sbom
* 将sbom上传到 Dependency-Track 服务器。默认情况下，这会上传 bom.xml 并在
* Dependency-Track 服务器上创建（或如果已存在则更新）一个项目
* 项目名称和版本映射到当前 Maven 项目 artifactId 和版本。如果您想覆盖项目名称或版本，请设置 projectName 或 projectVersion 属性
```mvn
mvn io.github.pmckeown:dependency-track-maven-plugin:upload-bom
```
## 获取存在漏洞的信息
* 在BOM上传之后，确定是否存在漏洞的最佳方式是使用findings目标，该目标在上传后即可使用。其他目标，如metrics和score，则会从Dependency-Track服务器异步拉取信息，因此在上传BOM后立即调用时可能无法获取。
* findings目标将打印出扫描中所有当前问题的详细信息，包括：组件细节、漏洞描述及抑制状态。
* findings目标可以配置为，如果在给定类别中发现的数量超过了该类别的阈值，则构建失败。
```mvn
mvn io.github.pmckeown:dependency-track-maven-plugin:findings
```
## 删除项目
* 从Dependency-Track服务器中删除当前项目或任何指定项目。
* 默认情况下不绑定到Maven生命周期的任何阶段。此目标可以随时独立运行，以删除项目。
* 预计使用场景是针对临时扫描短暂分支代码，一旦确定了评分，就需要从服务器中清除这些代码。
```mvn
mvn io.github.pmckeown:dependency-track-maven-plugin:delete-project
```
# syft 
## 安装
```shell
# GITHUB 地址下载 https://github.com/anchore/syft/releases/tag/v1.26.1
# 下载文件上传至服务器, 通过命令下载很容易卡死网络不通
# syft_1.26.1_linux_amd64.tar.gz

tar -xzvf syft_1.26.1_linux_amd64.tar.gz
sudo mv syft /usr/local/bin/
syft --version
```
## 容器生成sbom方式
```shell
syft 9b55d937a79b -o cyclonedx-xml > sbom.xml
```