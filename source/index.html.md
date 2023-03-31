---
title: Gaps and Islands

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - sql
  - markdown

toc_footers:
  - <a href='https:eeling//github.com/python-help'>Eeliang's Python help</a>
  - <a href'https://www.hackerrank.com/contests/crescent-practice-3rd-years/challenges/islands-1'>Try out the test questions at Hackerone</a>

includes:
  - dates_with_status
  - single_column
  - range_with_status
  - duplicate_group_error

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Gaps and Island problem

Hello reader!

I faced much difficulty when first encountering the [gaps and island problem](https://livebook.manning.com/book/sql-server-mvp-deep-dives/chapter-5/). Along with it's many variations and solution, it was tough to understand and tackle this problem.

So here is my take on the Gaps and Island problem (Eeliang style.)

### Challenge objectives
The goal of this series of challenges is to work with a continuous stream (dates or incremental counters) and identifying **islands** (overlaps or consecutive rows) and **gaps** (missing values or null values). 
