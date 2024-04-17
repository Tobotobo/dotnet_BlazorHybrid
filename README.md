# dotnet_BlazorHybrid

## 概要
* 途中
* .NET Blazor + Wpf

## やりたい事
* 開発環境をコンテナに移したいが素の WPF ではビューを作るのに XAML がネックになる。  
* 今後のことを考えるとデスクトップアプリオンリーではなく、Web の方にシフトしていきたい。
* ついでに言うと VSCode は XAML 開発ツールが貧弱で、実質 Visual Studio が必須なのもどうにかしたい。
* ビューとサービスは BlazorHybrid.RCL + BlazorHybrid.Web で開発する。  
  →　確認はこれからだが、上記の構成ならコンテナで開発できるはず  
  　　※サービスの別プロジェクトへの切り出しは今後の課題
* デスクトップアプリが必要な間は、開発はできるだけ上記で行いつつ、リリースは BlazorHybrid.RCL + BlazorHybrid.Wpf で行う。
* ビューが Web になるので Playwright など既存のビューのテストツールが活用できるようになるメリットもあると思われる。  
一方で、ビューが WebView2 のレンタリングに依存するようになるので、Web 開発者からするとブラウザの影響の考慮は当然だと思われるが、デスクトップアプリ開発者からすると、面倒事が増えたように感じるかもしれない。

## 参考サイト
神解説。ねこじょーかー様に多大な感謝を。  
このサイトが無かったら実現できなかった。

* [ねこじょーかー/Blazor HybridとBlazor Web AppのUIをRCLで共通化する手順](https://blazor-master.com/blazor-hybrid-maui-rcl/)
* [nekojoker/BlazorHybrid](https://github.com/nekojoker/BlazorHybrid)
    > .NET MAUI Blazor アプリと Blazor Web App の Razor コンポーネントや静的資産を Razor クラスライブラリで共通化したプロジェクトです。  
    > .NET 8 に対応しています。

## 環境
```
> dotnet --info   
.NET SDK:
 Version:           8.0.204   
 Commit:            c338c7548c
 Workload version:  8.0.200-manifests.c4df6daf

ランタイム環境:
 OS Name:     Windows
 OS Version:  10.0.19045
 OS Platform: Windows
 RID:         win-x64
 Base Path:   C:\Program Files\dotnet\sdk\8.0.204\
```

## BlazorHybrid.Wpf 追加

[Windows Presentation Foundation (WPF) の Blazor アプリを構築する](https://learn.microsoft.com/ja-jp/aspnet/core/blazor/hybrid/tutorials/wpf?view=aspnetcore-8.0)

css が読み込めずレイアウトが崩れていた問題は以下の修正で解消  
wwwroot/index.html  
```html
<head>
～～～～～～～～～～～～～～～
    <!-- <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
    <link href="css/app.css" rel="stylesheet" /> -->
    <link rel="stylesheet" href="_content/BlazorHybrid.RCL/bootstrap/bootstrap.min.css" />
    <link rel="stylesheet" href="app.css" />
～～～～～～～～～～～～～～～～
</head>
```

```
dotnet new wpf -o ./BlazorHybrid.Wpf
dotnet sln add ./BlazorHybrid.Wpf
dotnet add ./BlazorHybrid.Wpf reference ./BlazorHybrid.RCL
dotnet add ./BlazorHybrid.Wpf package Microsoft.AspNetCore.Components.WebView.Wpf --version 8.0.20
```

```
dotnet run --project ./BlazorHybrid.Wpf
dotnet watch --project ./BlazorHybrid.Wpf
```

exe 作成
```
dotnet publish ./BlazorHybrid.Wpf -r win-x64 --no-self-contained
dotnet publish ./BlazorHybrid.Wpf -r win-x64 --self-contained
```
※csproj に `<EnableWindowsTargeting>true</EnableWindowsTargeting>` の追加が必要

```
$ dotnet --version
8.0.103
```

ubuntu22.04 + 8.0.103 だとエラーになる？
```
$ dotnet publish -r win-x64 --no-self-contained
The imported project "/usr/lib/dotnet/sdk/8.0.103/Sdks/Microsoft.NET.Sdk.WindowsDesktop/targets/Microsoft.NET.Sdk.WindowsDesktop.targets" was not found. Confirm that the expression in the Import declaration ";/usr/lib/dotnet/sdk/8.0.103/Sdks/Microsoft.NET.Sdk/targets/../../Microsoft.NET.Sdk.WindowsDesktop/targets/Microsoft.NET.Sdk.WindowsDesktop.targets" is correct, and that the file exists on disk.  /usr/lib/dotnet/sdk/8.0.103/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.Sdk.targets
MSBuild version 17.8.5+b5265ef37 for .NET
  Determining projects to restore...
  Restored /vagrant/dotnet_BlazorHybrid/BlazorHybrid.RCL/BlazorHybrid.RCL.csproj (in 470 ms).
  Restored /vagrant/dotnet_BlazorHybrid/BlazorHybrid.Wpf/BlazorHybrid.Wpf.csproj (in 499 ms).
/usr/lib/dotnet/sdk/8.0.103/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.Sdk.targets(1355,3): error MSB4019: The imported project "/usr/lib/dotnet/sdk/8.0.103/Sdks/Microsoft.NET.Sdk.WindowsDesktop/targets/Microsoft.NET.Sdk.WindowsDesktop.targets" was not found. Confirm that the expression in the Import declaration ";/usr/lib/dotnet/sdk/8.0.103/Sdks/Microsoft.NET.Sdk/targets/../../Microsoft.NET.Sdk.WindowsDesktop/targets/Microsoft.NET.Sdk.WindowsDesktop.targets" is correct, and that the file exists on disk. [/vagrant/dotnet_BlazorHybrid/BlazorHybrid.Wpf/BlazorHybrid.Wpf.csproj]
```

8.0.204 だといける
```
$ $HOME/.dotnet/dotnet publish -r win-x64 --no-self-contained

Welcome to .NET 8.0!
---------------------
SDK Version: 8.0.204

Telemetry
---------
The .NET tools collect usage data in order to help us improve your experience. It is collected by Microsoft and shared with the community. You can opt-out of telemetry by setting the DOTNET_CLI_TELEMETRY_OPTOUT environment variable to '1' or 'true' using your favorite shell.

Read more about .NET CLI Tools telemetry: https://aka.ms/dotnet-cli-telemetry

----------------
Installed an ASP.NET Core HTTPS development certificate.
To trust the certificate, view the instructions: https://aka.ms/dotnet-https-linux

----------------
Write your first app: https://aka.ms/dotnet-hello-world
Find out what's new: https://aka.ms/dotnet-whats-new
Explore documentation: https://aka.ms/dotnet-docs
Report issues and find source on GitHub: https://github.com/dotnet/core
Use 'dotnet --help' to see available commands or visit: https://aka.ms/dotnet-cli
--------------------------------------------------------------------------------------
MSBuild version 17.9.8+b34f75857 for .NET
  Determining projects to restore...
  Restored /vagrant/dotnet_BlazorHybrid/BlazorHybrid.RCL/BlazorHybrid.RCL.csproj (in 594 ms).
  Restored /vagrant/dotnet_BlazorHybrid/BlazorHybrid.Wpf/BlazorHybrid.Wpf.csproj (in 1.58 sec).
  BlazorHybrid.RCL -> /vagrant/dotnet_BlazorHybrid/BlazorHybrid.RCL/bin/Release/net8.0/BlazorHybrid.RCL.dll
  BlazorHybrid.Wpf -> /vagrant/dotnet_BlazorHybrid/BlazorHybrid.Wpf/bin/Release/net8.0-windows/win-x64/BlazorHybrid.Wpf.dll
  BlazorHybrid.Wpf -> /vagrant/dotnet_BlazorHybrid/BlazorHybrid.Wpf/bin/Release/net8.0-windows/win-x64/publish/
```

## BlazorHybrid.Forms 追加

[Windows フォームの Blazor アプリを構築する](https://learn.microsoft.com/ja-jp/aspnet/core/blazor/hybrid/tutorials/windows-forms?view=aspnetcore-8.0)


```
dotnet new winforms -o BlazorHybrid.Forms
```
※csproj に `<EnableWindowsTargeting>true</EnableWindowsTargeting>` の追加が必要
```
dotnet sln add ./BlazorHybrid.Forms
dotnet add ./BlazorHybrid.Forms reference ./BlazorHybrid.RCL
dotnet add ./BlazorHybrid.Forms package Microsoft.AspNetCore.Components.WebView.WindowsForms --version 8.0.20
```

exe 作成
```
dotnet publish ./BlazorHybrid.Forms -r win-x64 --no-self-contained
dotnet publish ./BlazorHybrid.Forms -r win-x64 --self-contained
```

## TODO
* BlazorHybrid.Forms や BlazorHybrid.Wpf を Windows 環境で dotnet run すると起動はするが index.html の `<link rel="stylesheet" href="BlazorHybrid.XXXX.styles.css" />` が効かずレイアウトが崩れる。
