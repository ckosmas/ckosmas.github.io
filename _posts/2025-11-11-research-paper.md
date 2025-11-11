---
title: "Scrutinizing A Web-Based LLM in Private Browsing Mode: An Analysis of Memory Artifacts and Privacy Implications"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---
I recently completed a research paper that was published on the SANS Cybersecurity Whitepapers site. If you decide to read the full paper and find any aspects of it useful, have any ideas for further research, or want to provide any comments, please feel free to reach out to me directly.

**White Paper Link:** 
[https://www.sans.org/white-papers/scrutinizing-web-based-llm-private-browsing-mode-analysis-memory-artifacts-privacy-implications]

**Abstract:**
Using web-based LLMs such as ChatGPT has changed the web browsing landscape to become part of the typical everyday experience. Web browser memory forensics has been a staple technique used by investigators to gather information about a user's device and behavior, and to predict their preferences. The advent of private browsing furthered security and privacy while attempting to limit the collection of personally identifiable information. Private browsing has impacted the ability to gather complete data using typical browsing methods, such as surfing social media, shopping, visiting academic websites, downloading files, and logging into email accounts. Numerous studies have been conducted on memory-based forensics. These studies uncovered artifacts of interest to forensic analysts and provided privacy insights to the everyday user interested in safeguarding their data. This project aims to apply the same memory forensics techniques in previously explored browsing scenarios to a popular web-based LLM technology, ChatGPT. It attempts to determine if there are any artifacts worth exploring in more detail. It highlights the strength of privacy-focused browsers, such as Brave, compared to mass-market browsers, like Edge.

**Key Finding:**
It was possible to conduct memory forensics and reconstruct forensic artifacts of interest, including prompt and message content from various test cases in the web-based version of ChatGPT in Edge InPrivate mode.
