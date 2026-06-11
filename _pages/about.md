---
permalink: /
title: ""
excerpt: ""
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

{% if site.google_scholar_stats_use_cdn %}
{% assign gsDataBaseUrl = "https://cdn.jsdelivr.net/gh/" | append: site.repository | append: "@" %}
{% else %}
{% assign gsDataBaseUrl = "https://raw.githubusercontent.com/" | append: site.repository | append: "/" %}
{% endif %}
{% assign url = gsDataBaseUrl | append: "google-scholar-stats/gs_data_shieldsio.json" %}

<span class='anchor' id='about-me'></span>

I am a PhD student at <a href="http://silab.kaist.ac.kr/" style="color: #7289da; text-decoration: none;">System Intelligence</a> lab in KAIST. I am fortunate to be advised by Professor <a href="https://scholar.google.com/citations?user=sH2a0nkAAAAJ&hl=en" style="color: #7289da; text-decoration: none;">Jinkyoo Park</a>. Here is my <a href="/assets/CV_Jaewoo.pdf" class="link-in-list" style="color: #7289da; text-decoration: none;"> cv</a>.

My research interest lies in aligning generative models with human intention through the principles of reinforcement learning and probabilistic inference. I enjoy applying aligning techniques to real-world applications such as visual generation, robotics control, and scientific discovery.

If you would like to discuss research, explore collaborations, or just have a chat, please feel free to contact me anytime at <a href="mailto:jaewoo@kaist.ac.kr" style="color: #7289da; text-decoration: none;">jaewoo@kaist.ac.kr</a> — I am always happy to hear from you!


# 🔥 News
- 2026.06: &nbsp;🎉🎉 3 workshop papers accepted at ICML 2026 (SPIGM, DEMO, RLxF)
- 2026.01: &nbsp;🎉🎉 2 papers accepted at ICLR 2026
- 2026.01: &nbsp;🎉🎉 1 paper accepted at AISTATS 2026
- 2025.05: &nbsp;🎉🎉 1 paper accepted at ICML 2025
- 2024.09: &nbsp;🎉🎉 2 papers accepted at NeurIPS 2024

# 📖 Educations
- *2025.03 - current*, Ph.D in Industrial and Systems Engineering, KAIST
- *2023.03 - 2025.02*, M.S in Graduate School of AI, KAIST
- *2016.03 - 2022.02*, B.S in Mechanical Engineering, Sungkyunkwan University

# 💻 Working Experiences
- *2023.03 - current*, Cofounder & Research Engineer, MongooseAI
- *2022.02 - 2023.02*, Software Engineer, Samsung Electronics AI Center
- *2021.07 - 2021.08*, Intern Data Scientist, Samsung Electronics AI Center


# 📝 Publications 
(\*: Equal Contribution, †: Equal Corresponding Author) 

## Preprints

- **[P3] Aligning Few-Step Generative Models by Amortizing Sample-based Variational Inference**  
[[paper]](https://arxiv.org/abs/2605.26552) / [[code]](https://github.com/Jaewoopudding/FAV)  
**Jaewoo Lee**\*, Hyeongyu Kang\*, Dohyun Kim, Kyuil Sim, Woocheol Shin, Minsu Kim, Taeyoung Yun, Jeongjae Lee, Sanghyeok Choi, Tabitha Edith Lee, Jong Chul Ye†, Jinkyoo Park†  
<span style="color:darkorchid">**arXiv 2026**</span>  
<span style="color:darkorchid">**ICML 2026 SPIGM Workshop**</span>

- **[P2] Automated Kernel Discovery Towards Understanding High-dimensional Bayesian Optimization**  
[[paper]](https://arxiv.org/abs/2605.20249) / [[code]](https://github.com/Shin-woocheol/Kernel_discovery)  
Taeyoung Yun\*, Woocheol Shin\*, Inhyuck Song, **Jaewoo Lee**, Jinkyoo Park  
<span style="color:darkorchid">**arXiv 2026**</span>  
<span style="color:darkorchid">**ICML 2026 DEMO Workshop**</span>

- **[P1] Robust Exploration through Generative Replay**  
Inhyuck Song, Taeyoung Yun, **Jaewoo Lee**, Jinkyoo Park  
<span style="color:darkorchid">**ICML 2026 RLxF Workshop**</span>

## Conference Publications

- **[C6] Adaptive Replay Buffer for Offline-to-Online Reinforcement Learning**  
[[paper]](https://arxiv.org/abs/2512.10510) / [[code]](https://github.com/song970407/ARB)  
Chihyeon Song, **Jaewoo Lee**, Jinkyoo Park  
<span style="color:darkorchid">**AISTATS 2026**</span>

- **[C5] Diffusion Fine-Tuning via Reparameterized Policy Gradient of the Soft Q-Function**  
[[paper]](https://arxiv.org/abs/2512.04559) / [[code]](https://github.com/Shin-woocheol/SQDF)  
Hyeongyu Kang\*, **Jaewoo Lee**\*, Woocheol Shin\*, Kiyoung Om, Jinkyoo Park  
<span style="color:darkorchid">**ICLR 2026**</span>

- **[C4] Diffusion Alignment as Variational Expectation-Maximization**  
[[paper]](https://arxiv.org/abs/2510.00502) / [[code]](https://github.com/Jaewoopudding/dav)  
**Jaewoo Lee**, Minsu Kim, Sanghyeok Choi, Inhyuck Song, Sujin Yun, Hyeongyu Kang, Woocheol Shin, Taeyoung Yun, Kiyoung Om, Jinkyoo Park  
<span style="color:darkorchid">**ICLR 2026**</span>

- **[C3] Posterior Inference with Diffusion Models for High-dimensional Black-box Optimization**  
[[paper]](https://arxiv.org/abs/2502.16824) / [[code]](https://github.com/umkiyoung/DiBO)  
Taeyoung Yun\*, Kiyoung Om\*, **Jaewoo Lee**, Sujin Yun, Jinkyoo Park  
<span style="color:darkorchid">**ICML 2025**</span>  
<span style="color:darkorchid">**ICLR 2025 FPI Workshop**</span>

- **[C2] Guided Trajectory Generation with Diffusion Models for Offline Model-based Optimization**  
[[paper]](https://arxiv.org/abs/2407.01624) / [[code]](https://github.com/dbsxodud-11/GTG)  
Taeyoung Yun, Sujin Yun, **Jaewoo Lee**, Jinkyoo Park  
<span style="color:darkorchid">**NeurIPS 2024**</span>

- **[C1] GTA: Generative Trajectory Augmentation with Guidance for Offline Reinforcement Learning**  
[[paper]](https://arxiv.org/abs/2405.16907) / [[code]](https://github.com/Jaewoopudding/GTA)  
**Jaewoo Lee**\*, Sujin Yun\*, Taeyoung Yun, Jinkyoo Park  
<span style="color:darkorchid">**NeurIPS 2024**</span>  
<span style="color:darkorchid">**ICLR 2024 GenAI4DM Workshop (Spotlight)**</span>

# 🎖 Honors and Awards
- *2021.09* Dean's List, Sungkyunkwan University (honor for academic excellence)
- *2021.02* Excellence Award for Optimizing Shipbuilding Process Competition (Korea Shipbuilding and Offshore Engineering)
- *2020.09* Dean's List, Sungkyunkwan University (honor for academic excellence)

<!-- # 💬 Invited Talks
- *2021.06*, Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet. 
- *2021.03*, Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet.  \| [\[video\]](https://github.com/) -->
