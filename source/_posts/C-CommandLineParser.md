---
title: 'C# CommandLineParser'
date: 2019-10-03 00:07:31
tags: note
categories: note
---

## 下载：

https://www.nuget.org/packages/CommandLineParser/


## 使用：
```c#
    public class Options
    {
      [Option('i', "input", Required = true, HelpText = "help text")]
      public string input { get; set; }

      [Option('o', "output", Required = true, HelpText = "help text")]
      public string output { get; set; }
    }
    static void Main(string[] args)
    {
      Parser.Default.ParseArguments<Options>(args)
        .WithNotParsed(errs => HandleErrors(errs))
        .WithParsed(opts => RunOptions(opts));
    }
```
