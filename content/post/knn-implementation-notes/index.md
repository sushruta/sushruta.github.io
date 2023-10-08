---
title: "KNN Implementation Notes"
description: Discussion of the KNN part of Assignment 1 in CS231n
slug: knn-implementation-notes-cs231n
date: 2023-10-07T18:22:37-07:00
image: banner-image.png
categories:
  - CS231n
  - Machine Learning
tags:
  - cs231n
  - machine learning
---

### Introduction

This post documents my thoughts and ideas while implementing the K-Nearest Neighbour model for image classification. This is part of stanford's CS231n course. While the course is about deep learning and computer vision, we start off my implementing simple baseline models to compare our performance and track its evolution.

### Slow and Fast Code in Numpy

The assignment helped me understand the value of writing vectorized code in numpy.

#### What is Vectorized Code?

If there 
