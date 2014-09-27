---
layout: post
title: "RubyOnRails App on Docker: Part-II) Containerize App"
modified:
categories: articles
excerpt:
tags: ['RoR', 'Docker', 'Deploy', '12factor.net']
comments: true
share: true
image:
  feature:
date: 2014-09-27T16:27:46+05:30
---

RubyOnRails App On Docker: Part-II How are we doing?
===============

Index

- Setup and Install database
- Containerize RoR Apps
- Setup Reverse Proxy using Nginx



Setup and Install database
-------------------------

For this application we'll use MySQL. There are two ways of setting up MySQL 

[MySQL Dockerfile](https://github.com/dockerfile/mysql)