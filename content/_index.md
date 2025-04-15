---
title: ""
date: 2022-10-24
type: landing

design:
  spacing: "0rem"

sections:

  - block: resume-biography-3
    content:
      username: admin
      text: ""
      button:
        text: Download CV
        url: uploads/resume.pdf
      # Add another button below
      additional_buttons:
        - text: Contact Me
          url: "mailto:your-email@example.com"
        - text: LinkedIn
          url: "https://www.linkedin.com/in/your-profile"
    design:
      css_class: dark
      background:
        color: black
        image:
          filename: diego-ph-5LOhydOtTKU-unsplash-2.jpg
          filters:
            brightness: 1.0
          size: cover
          position: center
          parallax: false

  - block: markdown
    content:
      text: |
        <p style="font-size: 0.8rem; font-weight: normal; text-align: center; margin-top: 1rem; color: gray;">
          © {{ now.Format "2006" }} Ayantika Khanra · Built with <a href="https://github.com/HugoBlox/hugo-blox-builder" target="_blank" style="color: inherit; text-decoration: underline;">Hugo Blox</a>
        </p>
    design:
      spacing:
        padding: [1rem, 0, 0, 0]
---
