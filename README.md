# 用 PHP 解析 HTML

[![推广图](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/)

本指南将介绍三种在 PHP 中解析 HTML 的技术，并比较它们的优劣与差异：

- [用 PHP 解析 HTML](#用-php-解析-html)  
- [为什么要在 PHP 中解析 HTML？](#为什么要在-php-中解析-html)  
- [先决条件](#先决条件)  
- [在 PHP 中检索 HTML](#在-php-中检索-html)  
- [在 PHP 中解析 HTML：三种方法](#在-php-中解析-html三种方法)  
  - [方法一：使用 Dom\HTMLDocument](#方法一使用-domhtmldocument)  
  - [方法二：使用 Simple HTML DOM Parser](#方法二使用-simple-html-dom-parser)  
  - [方法三：使用 Symfony 的 DomCrawler 组件](#方法三使用-symfony-的-domcrawler-组件)  
- [在 PHP 中解析 HTML：对比表](#在-php-中解析-html对比表)  
- [结论](#结论)  

## 为什么要在 PHP 中解析 HTML？

在 PHP 中解析 HTML，指的是将 HTML 内容转换为其 DOM（[文档对象模型](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)）结构。一旦转换成 DOM 格式，就可以轻松地导航和操作 HTML 内容。

在 PHP 中解析 HTML 的主要原因包括：

- **数据提取**：从网页中获取特定内容，包括 HTML 元素中的文本或属性。  
- **自动化**：自动执行如内容抓取、报告生成和数据聚合等任务。  
- **服务器端 HTML 处理**：在渲染前对 HTML 进行清理、格式化或修改，以满足应用程序的需求。  

## 先决条件

在开始编码之前，请确认你已经在本地安装了 [PHP 8.4+](https://www.php.net/releases/8.4/en.php)。可使用以下命令进行验证：

```bash
php -v
```

输出应类似于：

```
PHP 8.4.3 (cli) (built: Jan 19 2025 14:20:58) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.4.3, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.3, Copyright (c), by Zend Technologies
```

接下来，初始化一个 Composer 项目来简化依赖管理。如果你的系统尚未安装 Composer，请[下载并按照安装说明操作](https://getcomposer.org/download/)。

首先，为你的 PHP HTML 项目创建一个新文件夹：

```bash
mkdir php-html-parser
```

在终端中进入该文件夹，并使用 `composer init` 命令初始化 Composer 项目：

```bash
composer init
```

在此过程中，你需要回答一些问题。默认答案通常已足够，但你也可以根据需要在此处进行更精细的配置，以便更好地适配你的 PHP HTML 解析项目。

接下来，在你喜欢的 IDE 中打开该项目文件夹。[带有 PHP 扩展的 Visual Studio Code](https://code.visualstudio.com/docs/languages/php) 或者 [IntelliJ WebStorm](https://www.jetbrains.com/webstorm/) 都是不错的选择。

现在，在项目文件夹中添加一个空的 `index.php` 文件。此时，你的项目结构应大致如下：

```
php-html-parser/
  ├── vendor/
  ├── composer.json
  └── index.php
```

打开 `index.php`，并添加以下代码来初始化项目：

```php
<?php

require_once __DIR__ . "/vendor/autoload.php";

// scraping logic...
```

使用下列命令运行脚本：

```bash
php index.php
```

## 在 PHP 中检索 HTML

在对 HTML 进行解析之前，需要先获取到相应的 HTML 内容。本节将展示两种在 PHP 中获取 HTML 的方法。我们也建议你阅读 [有关在 PHP 中进行网页抓取的指南](https://www.bright.cn/blog/how-tos/web-scraping-php)。

### 通过 CURL

PHP 原生支持 cURL，它是一个广泛使用的 HTTP 客户端，用于执行 HTTP 请求。可[启用 cURL 扩展](https://www.php.net/manual/en/book.curl.php)，或在 Ubuntu Linux 下使用以下命令安装：

```bash
sudo apt-get install php8.4-curl
```

你可以使用 cURL 向在线服务器发送 HTTP GET 请求，并获取由该服务器返回的 HTML 文档。以下示例脚本通过 GET 请求来获取 HTML 内容：

```bash
// initialize cURL session
$ch = curl_init();

// set the URL you want to make a GET request to
curl_setopt($ch, CURLOPT_URL, "https://www.scrapethissite.com/pages/forms/?per_page=100");

// return the response instead of outputting it
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

// execute the cURL request and store the result in $response
$html = curl_exec($ch);

// close the cURL session
curl_close($ch);

// output the HTML response
echo $html;
```

将上述代码片段添加到 `index.php` 并运行后，会输出类似以下内容的 HTML 代码：

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hockey Teams: Forms, Searching and Pagination | Scrape This Site | A public sandbox for learning web scraping</title>
    <link rel="icon" type="image/png" href="/static/images/scraper-icon.png" />
    <!-- Omitted for brevity... -->
</html>
```

### 从文件中读取

假设你有一个名为 `index.html` 的文件，其中包含了 [Scrape This Site](https://www.scrapethissite.com/pages/forms/?per_page=100) “Hockey Teams” 页面（先前用 cURL 抓取）的 HTML 内容：

![项目文件夹中的 index.html 文件](https://github.com/bright-cn/php-html-parsing/blob/main/Images/The-index.html-file-in-the-project-folder-2048x1216.png)

## 在 PHP 中解析 HTML：三种方法

本节将演示使用三个不同的库来解析 HTML：

1. 使用原生 PHP 的 `Dom\HTMLDocument`  
2. 使用 Simple HTML DOM Parser 库  
3. 使用 Symfony 的 `DomCrawler` 组件  

三种情况下，我们都会从本地文件 `index.html` 读取 HTML，选中页面上的所有冰球队数据并提取其信息：

![目标页面上的表格](https://github.com/bright-cn/php-html-parsing/blob/main/Images/The-table-on-the-target-page-2048x1107.png)

最终结果将是一个有关冰球队信息的列表，包含以下字段：

- 队伍名称（Team Name）  
- 年份（Year）  
- 胜场数（Wins）  
- 败场数（Losses）  
- 胜率（Win %）  
- 进球数（Goals For / GF）  
- 失球数（Goals Against / GA）  
- 净胜球（Goal Difference / Diff）  

这些数据来自于带有以下结构的 HTML 表格：

![表格行的 HTML DOM 结构](https://github.com/bright-cn/php-html-parsing/blob/main/Images/The-HTML-DOM-structure-of-the-tables-rows-2048x1134.png)

表格每一行中的各列都有特定的 class，可以通过这些 class 来选择对应的元素并获取其文本内容。

## 方法一：使用 Dom\HTMLDocument

PHP 8.4+ 原生支持 [`Dom\HTMLDocument`](https://www.php.net/manual/en/class.dom-htmldocument.php) 类。它表示一个 HTML 文档，并允许你解析 HTML 和遍历 DOM 树。

### 第一步：安装与设置

`Dom\HTMLDocument` 属于 [PHP 标准库（SPL）](https://www.php.net/manual/en/book.spl.php) 的一部分。但你需要启用 [DOM 扩展](https://www.php.net/manual/en/intro.dom.php) 或在 Linux 下使用以下命令安装：

```bash
sudo apt-get install php-dom
```

### 第二步：解析 HTML

可以使用如下方式解析 HTML 字符串：

```php
$dom = \DOM\HTMLDocument::createFromString($html);
```

对于本地文件 `index.html`，可以这样写：

```php
$dom = \DOM\HTMLDocument::createFromFile("./index.html");
```

解析后，`$dom` 将是一个 [`Dom\HTMLDocument`](https://www.php.net/manual/en/class.dom-htmldocument.php) 对象，你可以使用它提供的方法来进行数据提取。

### 第三步：数据提取

下面的示例演示如何使用 `\DOM\HTMLDocument` 获取所有冰球队数据：

```php
// select each row on the page
$table = $dom->getElementsByTagName("table")->item(0);
$rows = $table->getElementsByTagName("tr");

// iterate through each row and extract data
foreach ($rows as $row) {
  $cells = $row->getElementsByTagName("td");

  // extracting the data from each column
  $team = trim($cells->item(0)->textContent);
  $year = trim($cells->item(1)->textContent);
  $wins = trim($cells->item(2)->textContent);
  $losses = trim($cells->item(3)->textContent);
  $win_pct = trim($cells->item(5)->textContent);
  $goals_for = trim($cells->item(6)->textContent);
  $goals_against = trim($cells->item(7)->textContent);
  $goal_diff = trim($cells->item(8)->textContent);

  // create an array for the scraped team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print ("\n");
}
```

`\DOM\HTMLDocument` 并没有提供高级的查询方式，因此只能使用诸如 `getElementsByTagName()` 等方法来遍历元素或属性。

上述示例中使用的方法包括：

- `getElementsByTagName()`：获取文档中所有指定标签（如 `<table>`、`<tr>`、`<td>`）的元素；  
- `item()`：从返回的元素列表中获取单个元素；  
- `textContent`：获取元素内部的文本内容，用于提取可见的数据（例如球队名称、年份等）。  

此外还使用了 [`trim()`](https://www.php.net/manual/en/function.trim.php) 来去除空格，使提取得来的数据更干净。

当把以上代码添加到 `index.php` 文件后，将会输出类似如下的结果：

```php
Array
(
    [team] => Boston Bruins
    [year] => 1990
    [wins] => 44
    [losses] => 24
    [win_pct] => 0.55
    [goals_for] => 299
    [goals_against] => 264
    [goal_diff] => 35
)

// 省略若干输出...

Array
(
    [team] => Detroit Red Wings
    [year] => 1994
    [wins] => 33
    [losses] => 11
    [win_pct] => 0.688
    [goals_for] => 180
    [goals_against] => 117
    [goal_diff] => 63
) 
```

## 方法二：使用 Simple HTML DOM Parser

[Simple HTML DOM Parser](https://github.com/voku/simple_html_dom) 是一个轻量级的 PHP 库，可以让你更方便地解析和操作 HTML。

### 第一步：安装与设置

通过 Composer 安装 Simple HTML DOM Parser：

```bash
composer require voku/simple_html_dom
```

或者，你也可以手动下载并将 `simple_html_dom.php` 文件包含到项目中。

随后在 `index.php` 中引入：

```php
use voku\helper\HtmlDomParser;
```

### 第二步：解析 HTML

若要解析 HTML 字符串，可使用 `file_get_html()` 方法：

```php
$dom = HtmlDomParser::str_get_html($html);
```

在解析本地文件 `index.html` 时，可改用：

```php
$dom = HtmlDomParser::file_get_html($str);
```

这会将 HTML 内容加载到 `$dom` 对象中，你可以轻松地遍历 DOM 进行数据提取。

### 第三步：数据提取

使用 Simple HTML DOM Parser 来提取冰球队数据：

```php
// find all rows in the table
$rows = $dom->findMulti("table tr.team");

// loop through each row to extract the data
foreach ($rows as $row) {
  // extract data using CSS selectors
  $team_element = $row->findOne(".name");
  $team = trim($team_element->plaintext);

  $year_element = $row->findOne(".year");
  $year = trim($year_element->plaintext);

  $wins_element = $row->findOne(".wins");
  $wins = trim($wins_element->plaintext);

  $losses_element = $row->findOne(".losses");
  $losses = trim($losses_element->plaintext);

  $win_pct_element = $row->findOne(".pct");
  $win_pct = trim($win_pct_element->plaintext);

  $goals_for_element = $row->findOne(".gf");
  $goals_for = trim($goals_for_element->plaintext);

  $goals_against_element = $row->findOne(".ga");
  $goals_against = trim(string: $goals_against_element->plaintext);

  $goal_diff_element = $row->findOne(".diff");
  $goal_diff = trim(string: $goal_diff_element->plaintext);

  // create an array with the extracted team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print("\n");
}
```

上述 Simple HTML DOM Parser 中用到的功能包括：

- `findMulti()`：选中所有符合指定 CSS 选择器的元素；  
- `findOne()`：选中符合指定 CSS 选择器的第一个元素；  
- `plaintext`：用来获取 HTML 元素的原始文本内容。  

相比第一种方法，这次我们使用了更强大的 CSS 选择器。但最终数据提取的结果相同。

## 方法三：使用 Symfony 的 DomCrawler 组件

[Symfony 的 `DomCrawler` 组件](https://symfony.com/doc/current/components/dom_crawler.html) 提供了一种简单的方式来解析 HTML 文档，并从中提取所需数据。

> **注意**  
> 该组件属于 [Symfony 框架](https://symfony.com/) 的一部分，但可以单独使用。本节即是单独使用的示例。

### 第一步：安装与设置

使用 Composer 安装 Symfony 的 `DomCrawler` 组件：

```bash
composer require symfony/dom-crawler
```

随后在 `index.php` 文件中引入：

```php
use Symfony\Component\DomCrawler\Crawler;
```

### 第二步：解析 HTML

若要解析 HTML 字符串，可使用 `Crawler` 类的构造方法：

```php
$crawler = new Crawler($html);
```

若要解析本地文件，可以使用 `file_get_contents()` 并创建 `Crawler` 实例：

```php
$crawler = new Crawler(file_get_contents("./index.html"));
```

这样就把 HTML 内容加载到了 `$crawler` 对象中，你可以利用它提供的各种方法来遍历并提取数据。

### 第三步：数据提取

使用 `DomCrawler` 组件来提取冰球队信息：

```php
// select all rows within the table
$rows = $crawler->filter("table tr.team");

// loop through each row to extract the data
$rows->each(function ($row, $i) {
  // extract data using CSS selectors
  $team_element = $row->filter(".name");
  $team = trim($team_element->text());

  $year_element = $row->filter(".year");
  $year = trim($year_element->text());

  $wins_element = $row->filter(".wins");
  $wins = trim($wins_element->text());

  $losses_element = $row->filter(".losses");
  $losses = trim($losses_element->text());

  $win_pct_element = $row->filter(".pct");
  $win_pct = trim($win_pct_element->text());

  $goals_for_element = $row->filter(".gf");
  $goals_for = trim($goals_for_element->text());

  $goals_against_element = $row->filter(".ga");
  $goals_against = trim($goals_against_element->text());

  $goal_diff_element = $row->filter(".diff");
  $goal_diff = trim($goal_diff_element->text());

  // create an array with the extracted team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print ("\n");
});
```

`DomCrawler` 中用到的方法包括：

- `each()`：对选中的元素列表进行遍历；  
- `filter()`：使用 CSS 选择器选中所需元素；  
- `text()`：获取选中元素的文本内容。  

## 在 PHP 中解析 HTML：对比表

下表总结了上述三种 PHP HTML 解析方法的对比：

|                    | **\DOM\HTMLDocument** | **Simple HTML DOM Parser** | **Symfony 的 DomCrawler** |
| ------------------ | ---------------------- | -------------------------- | ------------------------- |
| **类型**           | 原生 PHP 组件         | 第三方库                   | Symfony 组件             |
| **GitHub Stars**   | —                     | 880+                       | 4,000+                   |
| **XPath 支持**     | ❌                    | ✔️                          | ✔️                        |
| **CSS 选择器支持** | ❌                    | ✔️                          | ✔️                        |
| **学习难度**       | 低                    | 低到中                     | 中                       |
| **易用度**         | 中                    | 高                          | 高                       |
| **API 丰富程度**   | 基础                  | 丰富                        | 丰富                     |

## 结论

以上方法在应对传统静态页面时非常有效。但是，如果目标网页依赖 JavaScript 渲染，那么简单的 HTML 解析方式（如上所示）就无法满足需求。这时，你需要使用带有更高级 HTML 解析功能的完整抓取浏览器，例如 [抓取浏览器](https://www.bright.cn/products/scraping-browser) 等。

如果你想绕过 HTML 解析，直接获取结构化数据，可尝试我们的 [成品数据集](https://www.bright.cn/products/datasets)，它覆盖了数百个网站！

立即创建 Bright Data 帐号，开始免费试用我们的数据和抓取方案吧！
