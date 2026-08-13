---
# Leave the homepage title empty to use the site title
title: ''
summary: ''
date: 2026-01-05
type: landing

sections:
  # Developer Hero - Gradient background with name, role, social, and CTAs
  - block: dev-hero
    id: hero
    content:
      username: me
      greeting: "Hi, I'm"
      show_status: true
      show_scroll_indicator: true
      typewriter:
        enable: true
        prefix: "I build"
        strings:
          - Rust + WebAssembly apps
          - self-hosted platforms
          - ML-powered software systems
          - open source software
        type_speed: 70
        delete_speed: 40
        pause_time: 2500
      cta_buttons:
        - text: View My Work
          url: "#projects"
          icon: arrow-down
        - text: Get In Touch
          url: "#contact"
          icon: envelope
    design:
      style: centered
      avatar_shape: circle
      animations: true
      background:
        color:
          light: "#fafafa"
          dark: "#0a0a0f"
      spacing:
        padding: ["6rem", "0", "4rem", "0"]


  
  # Filterable Portfolio - Alpine.js powered project filtering
  - block: portfolio
    id: projects
    content:
      title: "Featured Projects"
      subtitle: "A selection of my recent work"
      count: 0
      filters:
        folders:
          - projects
      buttons:
        - name: All
          tag: '*'
        - name: Full-Stack
          tag: Full-Stack
        - name: Frontend
          tag: Frontend
        - name: Backend
          tag: Backend
      default_button_index: 0
      # Archive link auto-shown if more projects exist than 'count' above
      # archive:
      #   enable: false  # Set to false to explicitly hide
      #   text: "Browse All"  # Customize text
      #   link: "/work/"  # Custom URL
    design:
      columns: 3
      background:
        color:
          light: "#ffffff"
          dark: "#0d0d12"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]
 
  - block: portfolio
    id: guides
    content:
      title: "Featured Guides"
      subtitle: "A selection of my recent work"
      count: 2
      filters:
        folders:
          - guides
      buttons:
        - name: All
          tag: '*'
        - name: Machine-Learning
          tag: Machine-Learning
        - name: Frontend
          tag: Frontend
        - name: Backend
          tag: Backend
        - name: Linux
          tag: Linux
      default_button_index: 0
      # Archive link auto-shown if more projects exist than 'count' above
      # archive:
      #   enable: false  # Set to false to explicitly hide
      #   text: "Browse All"  # Customize text
      #   link: "/work/"  # Custom URL
    design:
      columns: 2
      background:
        color:
          light: "#ffffff"
          dark: "#0d0d12"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]
 
  # Visual Tech Stack - Icons organized by category
  - block: tech-stack
    id: skills
    content:
      title: "Tech Stack"
      subtitle: "Technologies I use to build things"
      categories:
        - name: Languages
          items:
            - name: Python
              icon: devicon/python
            - name: Rust
              icon: devicon/rust
            - name: C++
              icon: devicon/cplusplus
            - name: Java 
              icon: devicon/java
        - name: Machine Learning
          items:
            - name: Keras
              icon: devicon/keras
            - name: scikitlearn
              icon: devicon/scikitlearn
            - name: PyTorch
              icon: devicon/pytorch
            - name: Ollama
              icon: devicon/ollama
        #- name: Backend
        #  items:
        #    - name: Node.js
        #      icon: devicon/nodejs
        #    - name: Express
        #      icon: devicon/express
        #    - name: PostgreSQL
        #      icon: devicon/postgresql
        #    - name: Redis
        #      icon: devicon/redis
        - name: DevOps
          items:
            - name: Docker
              icon: devicon/docker
            - name: Proxmox
              icon: devicon/proxmox
            - name: GitHub 
              icon: brands/github
            - name: Gitlab
              icon: devicon/gitlab
    design:
      style: grid
      show_levels: false
      background:
        color:
          light: "#f5f5f5"
          dark: "#08080c"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]
  
#  # Experience Timeline
#  - block: resume-experience
#    id: experience
#    content:
#      title: Experience
#      date_format: Jan 2006
#      items:
#        - title: Senior Software Engineer
#          company: Tech Corp
#          company_url: ''
#          company_logo: ''
#          location: San Francisco, CA
#          date_start: '2023-01-01'
#          date_end: ''
#          description: |2-
#            * Lead development of microservices architecture serving 1M+ users
#            * Improved API response time by 40% through optimization
#            * Mentored team of 5 junior developers
#            * Tech stack: React, Node.js, PostgreSQL, AWS
#        - title: Full-Stack Developer
#          company: Startup Inc
#          company_url: ''
#          company_logo: ''
#          location: Remote
#          date_start: '2021-06-01'
#          date_end: '2022-12-31'
#          description: |2-
#            * Built and deployed 3 production applications from scratch
#            * Implemented CI/CD pipeline reducing deployment time by 60%
#            * Collaborated with design team on UI/UX improvements
#            * Tech stack: Next.js, Express, MongoDB, Docker
#        - title: Junior Developer
#          company: Web Agency
#          company_url: ''
#          company_logo: ''
#          location: New York, NY
#          date_start: '2020-01-01'
#          date_end: '2021-05-31'
#          description: |2-
#            * Developed client websites using modern web technologies
#            * Maintained and updated legacy codebases
#            * Participated in code reviews and agile ceremonies
#            * Tech stack: React, WordPress, PHP, MySQL
#    design:
#      columns: '1'
#      background:
#        color:
#          light: "#ffffff"
#          dark: "#0d0d12"
#      spacing:
#        padding: ["4rem", "0", "4rem", "0"]
  
  # Recent Blog Posts
  - block: collection
    id: blog
    content:
      title: Recent Posts
      subtitle: 'Thoughts on web development, tech, and more'
      text: ''
      filters:
        folders:
          - blog
        exclude_featured: false
      count: 3
      order: desc
    design:
      view: card
      columns: 3
      background:
        color:
          light: "#f5f5f5"
          dark: "#08080c"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]
  
  # Contact Section
  - block: contact-info
    id: contact
    content:
      title: Get In Touch
      subtitle: "Let's build something amazing together"
      text: |-
        I'm always interested in hearing about new projects and opportunities.
        Whether you're looking to hire, collaborate, or just want to say hi, feel free to reach out!
      email: f.duwanoff@gmail.com
      autolink: true
    design:
      columns: '1'
      background:
        color:
          light: "#ffffff"
          dark: "#0d0d12"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]
  
  # CTA Card
  - block: cta-card
    content:
      title: "Open to Opportunities"
      text: |-

        I am currently looking for **Applied ML Engineering**, **MLOps**, and **Software Engineering** 
        roles focused on intelligent systems.
            </br></br> 

   #     Let's connect and discuss how I can help your team.
   #   button:
   #     text: 'Download Resume'
   #     url: uploads/resume.pdf
   #     new_tab: true
    design:
      card:
        # Light mode: soft pastel theme gradient | Dark mode: rich deep gradient
        css_class: 'bg-gradient-to-br from-primary-200 via-primary-100 to-secondary-200 dark:from-primary-600 dark:via-primary-700 dark:to-secondary-700'
        text_color: dark
      background:
        color:
          light: "#f5f5f5"
          dark: "#08080c"
      spacing:
        padding: ["4rem", "0", "6rem", "0"]
---
