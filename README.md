## 思维导航

* [前言](https://github.com)
* [项目介绍](https://github.com)
* [项目特性](https://github.com)
* [创建控制台应用](https://github.com)
* [安装 NuGet 包](https://github.com)
* [进行性能基准测试](https://github.com)
* [性能测试多种格式输出](https://github.com)
* [项目源码地址](https://github.com)
* [优秀项目和框架精选](https://github.com)

## 前言


在软件开发领域，性能基准测试是确保软件系统高效、稳定运行的重要环节。它可以帮助你评估应用程序的性能，了解其在不同条件下的响应时间、吞吐量、资源利用率等。通过基准测试，你可以确定系统在处理特定工作负载时的性能表现。


:[slower加速器](https://jisuanqi.org)## 项目介绍


BenchmarkDotNet是一个基于.NET开源、功能全面、易于使用的性能基准测试框架，它为.NET开发者提供了强大的性能评估和优化能力。通过自动化测试、多平台支持、高级统计分析和自定义配置等特性，BenchmarkDotNet帮助开发者更好地理解和优化软件系统的性能表现。


![](https://img2024.cnblogs.com/blog/1336199/202411/1336199-20241127002020436-1016987117.png)


## 项目特性


* 支持的语言：C\#、F\#、Visual Basic。
* 支持的操作系统：Windows、Linux、macOS。
* 支持的架构：x86、x64、ARM、ARM64、Wasm 和 LoongArch64。
* 支持的运行时：.NET 5\+、.NET Framework 4\.6\.1\+、.NET Core 3\.1\+、Mono、NativeAOT。


## 创建控制台应用


创建名为：`BenchmarkDotNetExercise`的.NET 9控制台应用。


![](https://img2024.cnblogs.com/blog/1336199/202411/1336199-20241127002033812-513734735.png)


![](https://img2024.cnblogs.com/blog/1336199/202411/1336199-20241127002039233-1318643499.png)


## 安装 NuGet 包


在NuGet包管理器中搜索：`BenchmarkDotNet` 包进行安装：


![](https://img2024.cnblogs.com/blog/1336199/202411/1336199-20241127002054435-2067804724.png)


## 进行性能基准测试


接下来我们对.NET中常见的三种加密哈希函数：`MD5`、`SHA256`、`SHA1`进行性能基准测试，来一起分析一下哪一种哈希算法性能更优、效率更快。


### HashFunctionsBenchmark



```
    [MemoryDiagnoser]//记录内存分配情况
    public class HashFunctionsBenchmark
    {
        private readonly string _inputData;

        public HashFunctionsBenchmark()
        {
            // 使用一个较长的字符串作为输入，以更好地反映哈希函数的性能
            _inputData = new string('y', 1000000);
        }

        [Benchmark]
        public byte[] MD5Hash()
        {
            using (MD5 md5 = MD5.Create())
            {
                return md5.ComputeHash(Encoding.UTF8.GetBytes(_inputData));
            }
        }

        [Benchmark]
        public byte[] SHA256Hash()
        {
            using (SHA256 sha256 = SHA256.Create())
            {
                return sha256.ComputeHash(Encoding.UTF8.GetBytes(_inputData));
            }
        }

        [Benchmark]
        public byte[] SHA1Hash()
        {
            using (SHA1 sha1 = SHA1.Create())
            {
                return sha1.ComputeHash(Encoding.UTF8.GetBytes(_inputData));
            }
        }
    }

```

### 运行基准测试



```
    internal class Program
    {
        static void Main(string[] args)
        {
            var summary = BenchmarkRunner.Run();
        }
    }

```

注意一定要设置为：`Release`模式运行，假如为`Debug`模式会提示下面异常：



```
// Validating benchmarks:
//    * Assembly BenchmarkDotNetExercise which defines benchmarks is non-optimized
Benchmark was built without optimization enabled (most probably a DEBUG configuration). Please, build it in RELEASE.
If you want to debug the benchmarks, please see https://benchmarkdotnet.org/articles/guides/troubleshooting.html#debugging-benchmarks.

```

![](https://img2024.cnblogs.com/blog/1336199/202411/1336199-20241127002118927-2026061094.png)


### 分析生成的报告


![](https://img2024.cnblogs.com/blog/1336199/202411/1336199-20241127002134106-880954710.png)


#### 说明：


* **Mean**: 所有测量值的算术平均值。
* **Error**: 99\.9% 置信区间的一半。
* **StdDev**: 所有测量值的标准差。
* **Gen0**: 第 0 代 GC 每 1000 次操作收集一次。
* **Gen1**: 第 1 代 GC 每 1000 次操作收集一次。
* **Gen2**: 第 2 代 GC 每 1000 次操作收集一次。
* **Allocated**: 每次操作分配的内存（仅托管内存，包含所有内容，1KB \= 1024B）。
* **1 ms**: 1 毫秒（0\.001 秒）。


#### 报告分析：




| Method | Mean | Error | StdDev | Gen0 | Gen1 | Gen2 | Allocated |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MD5Hash | 1\.952 ms | 0\.0169 ms | 0\.0158 ms | 197\.2656 | 197\.2656 | 197\.2656 | 976\.9 KB |
| SHA256Hash | 3\.907 ms | 0\.0157 ms | 0\.0147 ms | 195\.3125 | 195\.3125 | 195\.3125 | 976\.93 KB |
| SHA1Hash | 1\.780 ms | 0\.0231 ms | 0\.0193 ms | 197\.2656 | 197\.2656 | 197\.2656 | 976\.92 KB |


* MD5Hash 的平均耗时稍长于 SHA1Hash，但误差和标准差较小，性能稳定性较好，垃圾回收次数与 SHA1Hash 相同。
* SHA256Hash 的平均耗时最长，但误差和标准差最小，性能稳定性最好，垃圾回收次数略少于 MD5Hash 和 SHA1Hash。
* SHA1Hash 的平均耗时最短，但误差和标准差较大，表示其性能虽然优越但不太稳定。


## 性能测试多种格式输出


* MarkdownExporter：导出Markdown格式。
* AsciiDocExporter：导出AsciiDoc格式。
* HtmlExporter：导出HTML格式。
* CsvExporter：导出CSV（逗号分隔值）格式。
* RPlotExporter：导出R绘图文件格式。



```
[MarkdownExporter, AsciiDocExporter, HtmlExporter, CsvExporter, RPlotExporter]
public class HashFunctionsBenchmark
{
}

```


![](https://img2024.cnblogs.com/blog/1336199/202411/1336199-20241127002158292-1942909670.png)


## 项目源码地址


更多项目实用功能和特性欢迎前往项目开源地址查看👀，别忘了给项目一个Star支持💖。


* 开源地址：[https://github.com/dotnet/BenchmarkDotNet](https://github.com)
* 文章示例：[https://github.com/YSGStudyHards/DotNetExercises/tree/master/BenchmarkDotNetExercise](https://github.com)


## 优秀项目和框架精选


该项目已收录到C\#/.NET/.NET Core优秀项目和框架精选中，关注优秀项目和框架精选能让你及时了解C\#、.NET和.NET Core领域的最新动态和最佳实践，提高开发工作效率和质量。坑已挖，欢迎大家踊跃提交PR推荐或自荐（让优秀的项目和框架不被埋没🤞）。


* GitHub开源地址：[https://github.com/YSGStudyHards/DotNetGuide/blob/main/docs/DotNet/DotNetProjectPicks.md](https://github.com)
* Gitee开源地址：[https://gitee.com/ysgdaydayup/DotNetGuide/blob/main/docs/DotNet/DotNetProjectPicks.md](https://github.com)



