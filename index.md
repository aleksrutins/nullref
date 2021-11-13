---
layout: landing
header:
    title: The Null Reference
    slogan: Tips & Tricks for FTC Coding
    logo: /assets/Logo.svg
    buttons:
        - title: Contribute
          url: 'https://github.com/aleksrutins/nullref'
title: "Welcome"
toc: true
---
# Welcome!
Welcome to The Null Reference. This is very much a work in progress. It's powered by Jekyll and [Inkwell](https://github.com/aleksrutins/inkwell), and hosted on GitHub at <https://github.com/aleksrutins/nullref>. Please contribute your tips and tricks about FTC coding!

## Contributing
First, [learn Markdown](https://www.markdowntutorial.com).

To add a page, add a Markdown file to the root of the repo, and follow this template:
```markdown
---
layout: docpage
title: "[page title]"
author: Your Name
---
# Page Title
[content here]
```
Then, add it to the navigation. Open `_data/navigation.yml`, and add an entry to the section that the page should be part of that looks like this:
```yaml
- name: # page title
  url: # page slug (filename without the .md extension)
```

To modify a page, open that page, and write your modifications!

Then, create a pull request on GitHub, and I will review it.