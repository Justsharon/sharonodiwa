---
layout: default
title: Home
---

<section class="hero">
 <div class="hero-text">
  <h2>Hi, I'm <span class="highlight">Sharon Odiwa</span></h2>
  <p class="tagline">
    Data Analyst with 3 years of engineering experience. 
    I bring technical depth to analysis work — clean pipelines, 
    reproducible code, and dashboards that actually get used.
  </p>
  <div class="hero-actions">
  <a href="{{ "/contact/" | relative_url }}" class="btn btn-outline">Get In Touch</a>
</div>
  </div>
  <div class="hero-photo">
    <img
      src="{{ '/assets/images/sharonlogo.jpg' | relative_url }}"
      alt="Sharon Odiwa"
      onerror="this.src='https://ui-avatars.com/api/?name=Sharon+Odiwa&size=300&background=1a2e2a&color=ffffff&rounded=true&bold=true&font-size=0.35'"
    />
  </div>
</section>

<section class="skills-section">
  <h2>What I Work With</h2>
  <div class="skills-grid">
    <div class="skill-block">
      <h4>Data & Querying</h4>
      <p>SQL (PostgreSQL, MySQL), Excel, Google Sheets</p>
    </div>
    <div class="skill-block">
      <h4>Analysis & Modeling</h4>
      <p>Python (Pandas, NumPy, Scikit-learn), Statistics, A/B Testing</p>
    </div>
    <div class="skill-block">
      <h4>Visualization</h4>
      <p>Power BI, Matplotlib, Seaborn</p>
    </div>
    <div class="skill-block">
      <h4>Workflow</h4>
      <p>Git, Jupyter Notebooks, VS Code, MySQL Workbench</p>
    </div>
  </div>
</section>

<section class="featured-section">
  <h2>Featured Projects</h2>
  {% assign featured = site.projects | limit: 3 %}
  {% for project in featured %}
    {% include project-card.html project=project %}
  {% endfor %}
  <p style="text-align:center; margin-top: 2rem;">
    <a href="{{ "/projects/" | relative_url }}" class="btn">See All Projects →</a>
  </p>
</section>
