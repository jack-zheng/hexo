---
title: How to detect consecutive integers in a list
date: 2021-05-10 18:30:51
categories:
- python
- java
tags:
- 查找
---

在解析文件的时候遇到一个问题，有一个 sourcegraph 的结果集合，包含一些方法，大致格式如下

```json
{
    "lineMatches": [
                {
                  "preview": "  public void createUserInfoElement(ECFieldConfig ecf, MDFEntityBuffer<EPUserInfoConfig> buffer, SyncValueObject svo,\r",
                  "lineNumber": 844
                },
                {
                  "preview": "      List<EPUserInfoElementHrisSyncMappingConfig> userInfoMappingList, SuccessionDataModelDelegate sdmBean)\r",
                  "lineNumber": 845
                },
                {
                  "preview": "    public boolean syncData(IHrisElementField hf, List<Locale> supportedLocales, List<SyncValueObject> svoList, SuccessionDataModelDelegate sdm) {\r",
                  "lineNumber": 2277
                }
              ]
}
```

我想要的处理效果是，将 lineNumber 连续的对象整合在一起，需要怎么做？