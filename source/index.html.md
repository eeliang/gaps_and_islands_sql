---
title: Gaps and Islands

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - sql

toc_footers:
  - <a href='https://eeliang.github.io/python-help'>Eeliang's Python help</a>
  - <a href='https://www.hackerrank.com/challenges/sql-projects/problem?isFullScreen=false'>Try test questions at HackerOne</a>

includes:
  - dates_with_status
  - single_column
  - range_with_status
  - unique_group_keys_from_dense_rank
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
The goal of this challenge is to work with on a continuous datatype (dates or incremental counters) to identify: 

- **islands** (overlaps or consecutive rows) and,
- **gaps** (missing values or null values). 
