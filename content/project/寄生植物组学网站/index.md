---
title: 寄生植物组学网站的开发和维护
summary: 基于Django框架
tags:
  - Django
date: '2023-07-16T00:00:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

image:
  caption: 网站截图
  focal_point: Smart

links:
  - icon: browser
    icon_pack: fa
    name: Jump to
    url: http://210.72.89.18:7382/
  - icon: github
    icon_pack: fab
    name: source code
    url: https://github.com/fromYuechenimportQi/ParasiticPlantBase
url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
---

目前，寄生植物组学信息网站共包含5个物种的信息，即菟丝子、天麻、埃及列当、向日葵列当和钟萼草。

在功能方面，网站共有3个模块：基因查询、基因可视化(Jbrowse)和基因序列比对(Blast)。

基因信息可以通过基因ID和基因Name进行检索，此信息包括基因序列、在染色体上的位置、以及转录本的所有信息(5'UTR, CDS, 3'UTR和intron等)，并且包含跳转到Jbrowse进行基因可视化的接口。

基因可视化调用<a href='https://jbrowse.org/jb2/'>Jbrowse2包</a>，该包根据后端提供的genome.fasta和genome.gff文件，通过React框架直接在前端进行可视化。

基因比对使用NCBI的BLAST+软件，比对结果输出XML格式文件，经解析后在前端进行渲染。

