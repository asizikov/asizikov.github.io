---
date: "2023-02-22T00:00:00Z"
title: Integration tests for .NET CLI tools
tags: 
 - github-actions
 - dotnet 
slug: dotnet-cli-integration-tests
---

I've spent Saturday evening working on a small project. I've built a .NET clone of `tree` utitly. You know, the one that renders a nice tree view for a given directory.

The project is available on GitHub under the [dtree](https://github.com/asizikov/dtree) name. It's a .NET global tool that you can install with the following command:

```bash
dotnet tool install -g dtree
```

The tool is pretty simple. It takes a path to a directory as an argument and renders a tree view for it. Here is an example:

```bash
dtree -a
📁 .
├── 📁 .devcontainer
│   └── devcontainer.json
├── 📁 .github
│   └── 📁 workflows
│      └── build-and-publish.yml
├── 📁 src
│   ├── dtree.csproj
│   ├── Program.cs
│   └── TreeNode.cs
├── .gitignore
├── LICENSE
└── README.md
```

The codebase is quite hacky. It's a single file with top level statements. Not much abstractions over the `System.Console` and `System.IO.Directory` libraries. However, I wanted to make sure that it works as expected. So, I've added a few integration tests to the project.

I wanted to build tests that run the compiled tool and compare the output with the expected result. One would expect to have something similar to ASP.NET [WebApplicationFactory](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1?view=aspnetcore-7.0). However, there is no such thing for .NET CLI tools. So, I had to come up with a solution.

I've created a new project, that won't reference the `dtree` project directly. Instead, it will use the `CliRunner` wrapper around `Process` object that will execute `dotnet run` command to compile and run the `dtree` tool.

```csharp

using System.Diagnostics;

namespace dtree.IntegrationTests;

public class CliRunner
{
    private readonly string cliProjectPath;
    private readonly Process process;

    public CliRunner(string cliProjectPath)
    {
        this.cliProjectPath = cliProjectPath;

        process = new Process();
        process.StartInfo.FileName = "dotnet";
        process.StartInfo.UseShellExecute = false;
        process.StartInfo.RedirectStandardOutput = true;
    }

    public (int exitCode, string output) Run(string args)
    {
        process.StartInfo.Arguments = $"run --project {cliProjectPath} -- {args}";

        process.Start();
        var output = process.StandardOutput.ReadToEnd();

        process.WaitForExit();

        return (process.ExitCode, output);
    }
}
```

And this wrapper can be used in tests.

```csharp
using Xunit.Abstractions;

namespace dtree.IntegrationTests;

public class DTreeTests
{
    private readonly string cliProjectPath;
    private readonly CliRunner runner;

    public DTreeTests(ITestOutputHelper output)
    {
        cliProjectPath = Path.Combine(Directory.GetCurrentDirectory(), "../../../../src/dtree.csproj");
        output.WriteLine($"{cliProjectPath}");
        runner = new CliRunner(cliProjectPath);
    }

    [Theory]
    [InlineData("-d")]
    [InlineData("--max-depth")]
    public void Negative_Depth_Results_As_Error(string key)
    {
        var (exitCode, output) = runner.Run($"{key} -1");

        Assert.Equal($"😱 Max depth must be greater than 0{Environment.NewLine}", output);
        Assert.Equal(1, exitCode);
    }

    [Theory]
    [InlineData("-a")]
    [InlineData("--all")]
    public void All_Files_Should_Be_Included(string key)
    {
       var (exitCode, output) = runner.Run($"{key} -p ../../../../");

        Assert.Contains("📁 .devcontainer", output);
        Assert.Contains("📁 .github", output);
        Assert.Contains("📁 src", output);
        Assert.Contains(".gitignore", output);
        Assert.Contains("LICENSE", output);
        Assert.Contains("README.md", output);
        Assert.Equal(0, exitCode);
    }
}
```

This way I can run the tests in CI

```yaml
    steps:
      - uses: actions/checkout@v3
        name: 📥 Checkout
      - name: 🧪 Execute Unit Tests
        run: dotnet test ./tests
```

And be sure to publish the package to [NuGet](https://www.nuget.org/packages/dtree) if everything goes well.

Give it a try 🙂