---
# Leave the homepage title empty to use the site title
title: ""
date: 2022-10-24
type: landing

design:
  spacing: "6rem"

sections:
  - block: resume-biography-3
    content:
      username: admin
      text: ""  # This can remain empty, but ensure it's properly formatted
      button:
        text: Download CV
        url: uploads/resume.pdf
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

  # Footer text section with inline styling
  - block: markdown
    content:
      text: |
        <p style="font-size: 0.8rem; font-weight: normal; text-align: center; margin-top: 1rem;">
          © 2025 Ayantika Khanra · Built with <a href="https://github.com/HugoBlox/hugo-blox-builder" target="_blank" style="color: inherit; text-decoration: underline;">Hugo Blox</a>
        </p>
    design:
      spacing:
        padding: [1rem, 0, 0, 0]
---
