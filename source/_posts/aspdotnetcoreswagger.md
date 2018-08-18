title: swagger生成xml文件上传到azure云主机
date: 2017-12-20 15:25:04
tags: [Asp.Net Core, Swagger, azure云]
---

swagger xml文件默认不上传到azure云主机，如需要上传需要修改 **csproj** 文件（Asp.Net Core环境）。添加如下配置：

    <PropertyGroup>
      <GenerateDocumentationFile>true</GenerateDocumentationFile>
    </PropertyGroup>
    
    <Target Name="IncludeDocFile" BeforeTargets="PrepareForPublish">
      <ItemGroup Condition=" '$(DocumentationFile)' != '' ">
        <_DocumentationFile Include="$(DocumentationFile)" />
        <ContentWithTargetPath Include="@(_DocumentationFile->'%(FullPath)')"
                               RelativePath="%(_DocumentationFile.Identity)"
                               TargetPath="%(_DocumentationFile.Filename)%(_DocumentationFile.Extension)"
                               CopyToPublishDirectory="PreserveNewest" />
      </ItemGroup>
    </Target>


[issue: dotnet/sdk/issues/795](https://github.com/dotnet/sdk/issues/795)




