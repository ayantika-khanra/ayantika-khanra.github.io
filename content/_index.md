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

  # Footer text section with custom CSS styling
  - block: markdown
    content:
      text: |
        © 2025 Ayantika Khanra · Built with [Hugo Blox](https://github.com/HugoBlox/hugo-blox-builder)
    design:
      css_class: "small-text"  # Apply custom class
      spacing:
        padding: [1rem, 0, 0, 0]  # Reduce space under the image

# If you have sections below this point that should be removed, comment them out properly:
# - Research section
# - Publications
# - Talks
# - News
# - Call to Action (CTA) card
---
