.
├── .gitattributes
├── .github
│   ├── dependabot.yml
│   └── workflows
│       └── build.yml
├── .gitignore
├── Karls.Templates.csproj
├── LICENSE
├── package.json
├── README.md
├── templates
│   └── opinionated-solution
│       ├── BASE_NAME.sln
│       ├── .config
│       │   └── dotnet-tools.json
│       ├── Directory.Build.props
│       ├── .editorconfig
│       ├── .gitattributes
│       ├── .github
│       │   ├── dependabot.yml
│       │   └── workflows
│       │       ├── build.yml
│       │       ├── combine-dependabot-prs.yml
│       │       └── dependabot-build.yml
│       ├── .gitignore
│       ├── nuget.config
│       ├── omnisharp.json
│       ├── package.json
│       ├── README.md
│       ├── src
│       │   ├── BannedSymbols.txt
│       │   ├── BASE_NAME.Core
│       │   │   ├── BASE_NAME.Core.csproj
│       │   │   ├── Common
│       │   │   │   └── HttpClientBase.cs
│       │   │   └── Exceptions
│       │   │       ├── HttpBadRequestException.cs
│       │   │       ├── HttpExceptionBase.cs
│       │   │       ├── HttpInternalServerErrorException.cs
│       │   │       ├── HttpNotFoundException.cs
│       │   │       ├── HttpServiceUnavailableException.cs
│       │   │       ├── HttpStatusCodeException.cs
│       │   │       └── HttpUnauthorizedException.cs
│       │   └── Directory.Build.props
│       ├── .template.config
│       │   └── template.json
│       ├── test
│       │   ├── BASE_NAME.Core.Tests
│       │   │   ├── BASE_NAME.Core.Tests.csproj
│       │   │   └── UnitTest1.cs
│       │   ├── BASE_NAME.TestHelpers
│       │   │   ├── AssemblyInfo.cs
│       │   │   ├── BASE_NAME.TestHelpers.csproj
│       │   │   ├── Http
│       │   │   │   ├── FakeHttpContent.cs
│       │   │   │   ├── FakeHttpMessageHandler.cs
│       │   │   │   ├── FuncHttpMessageHandler.cs
│       │   │   │   └── HttpClientActivator.cs
│       │   │   └── Resources.cs
│       │   └── Directory.Build.props
│       └── .vscode
│           └── settings.json
└── .vscode
    └── settings.json
