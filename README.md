# 🚀 Hướng dẫn cài đặt Seq & tích hợp ASP.NET Core

## 📑 Mục lục
- [1. Giới thiệu Seq](#1-giới-thiệu-seq)
- [2. Cài đặt Seq](#2-cài-đặt-seq)  
  - [2.1 Windows](#21-windows)  
  - [2.2 Docker](#23-docker)  
- [3. Tích hợp Seq với ASP.NET Core](#3-tích-hợp-seq-với-aspnet-core)  
  - [3.1 Cài đặt NuGet](#31-cài-đặt-nuget)  
  - [3.2 Cấu hình appsettingsjson](#32-cấu-hình-appsettingsjson)  
  - [3.3 Programcs](#33-programcs)  
  - [3.4 Ví dụ log trong Controller](#34-ví-dụ-log-trong-controller)  
- [4. Truy cập Seq](#4-truy-cập-seq)

---

## 1. Giới thiệu Seq
- **Seq** là log server giúp thu thập, phân tích và trực quan hóa log ứng dụng.
- Thường dùng với **Serilog** trong .NET, hỗ trợ query log mạnh mẽ (gần giống SQL).
- Website: [https://datalust.co/seq](https://datalust.co/seq)

## 2. Cài đặt Seq
### 2.1 Windows 
1. Tải bộ cài tại: [https://datalust.co/download](https://datalust.co/download)  
2. Cài đặt vào thư mục mong muốn, ví dụ: D:\Program Files\Seq
3. Sau khi cài xong, thêm đường dẫn vào **Environment Variables**
   - `PATH`: D:\Program Files\Seq\
   - `PATH`: D:\Program Files\Seq\Client\
5. Mặc định chạy tại: [http://localhost:5341](http://localhost:5341). 

### 2.2 Cài đặt bằng Docker
Chạy container Seq với lệnh sau:

```bash
docker run -d --name seq \
  -e ACCEPT_EULA=Y \
  -p 5341:80 \
  datalust/seq:latest
```

## 3. Tích hợp Seq với ASP.NET Core
Các file logs được lưu vào wwwroot/Logs
### 3.1 Cài đặt NuGet
Cài các package cần thiết:

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Seq
dotnet add package Serilog.Sinks.File
```
### 3.2 Cấu hình appsettings.json
```bash
"Serilog": {
  "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File", "Serilog.Sinks.Seq" ],
  "MinimumLevel": {
    "Default": "Information",
    "Override": {
      "Microsoft": "Warning",
      "Microsoft.AspNetCore": "Information",
      "System": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning"
    }
  },
  "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ],
  "Properties": {
    "AppName": "my-aspnet-app"
  },
  "WriteTo": [
    { "Name": "Console" },
    {
      "Name": "File",
      "Args": {
        "path": "Logs/log-.txt",
        "rollingInterval": "Day",
        "retainedFileCountLimit": 30,
        "shared": true
      }
    },
    {
      "Name": "Seq",
      "Args": {
        "serverUrl": "http://localhost:5341"
      }
    }
  ]
}

```
### 3.3 Program.cs
```bash
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Đọc config từ appsettings.json
builder.Host.UseSerilog((context, services, configuration) =>
    configuration.ReadFrom.Configuration(context.Configuration)
                 .ReadFrom.Services(services)
                 .Enrich.FromLogContext());

var app = builder.Build();

app.MapControllers();
app.Run();

```
### 3.4 Ví dụ log trong Controller
```bash
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

[ApiController]
[Route("[controller]")]
public class TestController : ControllerBase
{
    private readonly ILogger<TestController> _logger;

    public TestController(ILogger<TestController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Get()
    {
        _logger.LogInformation("Hello from ASP.NET Core with Seq!");
        return Ok("Log đã được ghi vào Seq");
    }
}

```

