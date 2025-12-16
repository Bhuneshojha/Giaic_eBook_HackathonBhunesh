# Docusaurus/Spec-Kit Plus Deployment Setup for Physical AI & Humanoid Robotics Book

## Overview

This document provides comprehensive instructions for setting up and deploying the Physical AI & Humanoid Robotics book using Docusaurus and Spec-Kit Plus. The setup will create a professional, searchable, and navigable documentation site hosted on GitHub Pages.

## Prerequisites

- Node.js (version 18 or higher)
- npm or yarn package manager
- Git
- GitHub account

## 1. Project Structure

First, let's establish the recommended project structure:

```
physical-ai-humanoid-robotics-book/
├── docs/
│   ├── intro.md
│   ├── ros2/
│   │   ├── index.md
│   │   └── concepts.md
│   ├── gazebo-unity/
│   │   ├── index.md
│   │   └── simulation.md
│   ├── nvidia-isaac/
│   │   ├── index.md
│   │   └── ai-integration.md
│   └── vla-capstone/
│       ├── index.md
│       └── implementation.md
├── src/
│   ├── components/
│   ├── css/
│   └── pages/
├── static/
│   ├── img/
│   └── files/
├── docusaurus.config.js
├── package.json
├── sidebars.js
└── README.md
```

## 2. Initialize Docusaurus Project

### 2.1 Create New Docusaurus Project

```bash
# Create a new Docusaurus project
npx create-docusaurus@latest physical-ai-humanoid-robotics-book classic

cd physical-ai-humanoid-robotics-book
```

### 2.2 Install Additional Dependencies

```bash
npm install @docusaurus/module-type-aliases @docusaurus/types
npm install @docusaurus/preset-classic
npm install @mdx-js/react
npm install clsx
npm install prism-react-renderer
```

## 3. Configuration Files

### 3.1 docusaurus.config.js

```javascript
// docusaurus.config.js
import {themes as prismThemes} from 'prism-react-renderer';

/** @type {import('@docusaurus/types').Config} */
const config = {
  title: 'Physical AI & Humanoid Robotics',
  tagline: 'A comprehensive academic book on modern robotics systems',
  favicon: 'img/favicon.ico',

  // Set the production url of your site here
  url: 'https://your-username.github.io',
  // Set the /<baseUrl>/ pathname under which your site is served
  // For GitHub pages deployment, it is often '/<projectName>/'
  baseUrl: '/physical-ai-humanoid-robotics-book/',

  // GitHub pages deployment config.
  organizationName: 'your-username', // Usually your GitHub org/user name.
  projectName: 'physical-ai-humanoid-robotics-book', // Usually your repo name.
  deploymentBranch: 'gh-pages',
  trailingSlash: false,

  onBrokenLinks: 'throw',
  onBrokenMarkdownLinks: 'warn',

  // Even if you don't use internationalization, you can use this field to set
  // useful metadata like html lang. For example, if your site is Chinese, you
  // may want to replace "en" with "zh-Hans".
  i18n: {
    defaultLocale: 'en',
    locales: ['en'],
  },

  presets: [
    [
      'classic',
      /** @type {import('@docusaurus/preset-classic').Options} */
      ({
        docs: {
          sidebarPath: './sidebars.js',
          // Please change this to your repo.
          // Remove this to remove the "edit this page" links.
          editUrl:
            'https://github.com/your-username/physical-ai-humanoid-robotics-book/tree/main/',
        },
        blog: {
          showReadingTime: true,
          // Please change this to your repo.
          // Remove this to remove the "edit this page" links.
          editUrl:
            'https://github.com/your-username/physical-ai-humanoid-robotics-book/tree/main/',
        },
        theme: {
          customCss: './src/css/custom.css',
        },
      }),
    ],
  ],

  themeConfig:
    /** @type {import('@docusaurus/preset-classic').ThemeConfig} */
    ({
      // Replace with your project's social card
      image: 'img/docusaurus-social-card.jpg',
      navbar: {
        title: 'Physical AI & Humanoid Robotics',
        logo: {
          alt: 'Physical AI & Humanoid Robotics Logo',
          src: 'img/logo.svg',
        },
        items: [
          {
            type: 'docSidebar',
            sidebarId: 'tutorialSidebar',
            position: 'left',
            label: 'Book',
          },
          {
            href: 'https://github.com/your-username/physical-ai-humanoid-robotics-book',
            label: 'GitHub',
            position: 'right',
          },
        ],
      },
      footer: {
        style: 'dark',
        links: [
          {
            title: 'Book Sections',
            items: [
              {
                label: 'ROS 2 - Robotic Nervous System',
                to: '/docs/ros2',
              },
              {
                label: 'Gazebo/Unity - Digital Twin',
                to: '/docs/gazebo-unity',
              },
              {
                label: 'NVIDIA Isaac - AI Robot Brain',
                to: '/docs/nvidia-isaac',
              },
              {
                label: 'VLA & Capstone Project',
                to: '/docs/vla-capstone',
              },
            ],
          },
          {
            title: 'Community',
            items: [
              {
                label: 'GitHub',
                href: 'https://github.com/your-username/physical-ai-humanoid-robotics-book',
              },
            ],
          },
          {
            title: 'More',
            items: [
              {
                label: 'Chatbot',
                href: '/chatbot', // We'll create this page later
              },
              {
                label: 'GitHub',
                href: 'https://github.com/your-username/physical-ai-humanoid-robotics-book',
              },
            ],
          },
        ],
        copyright: `Copyright © ${new Date().getFullYear()} Physical AI & Humanoid Robotics Book. Built with Docusaurus.`,
      },
      prism: {
        theme: prismThemes.github,
        darkTheme: prismThemes.dracula,
        additionalLanguages: ['python', 'cpp', 'bash', 'docker', 'json', 'yaml'],
      },
    }),
};

export default config;
```

### 3.2 sidebars.js

```javascript
// sidebars.js
/** @type {import('@docusaurus/plugin-content-docs').SidebarsConfig} */
const sidebars = {
  tutorialSidebar: [
    {
      type: 'autogenerated',
      dirName: '.', // Generate sidebar from docs folder structure
    },
  ],
};

export default sidebars;
```

### 3.3 Custom CSS (src/css/custom.css)

```css
/* src/css/custom.css */
/**
 * Any CSS included here will be global. The classic template
 * bundles Infima by default. Infima is a CSS framework designed to
 * work well for content-centric websites.
 */

/* You can override the default Infima variables here. */
:root {
  --ifm-color-primary: #2e8555;
  --ifm-color-primary-dark: #29784c;
  --ifm-color-primary-darker: #277148;
  --ifm-color-primary-darkest: #205d3b;
  --ifm-color-primary-light: #33925d;
  --ifm-color-primary-lighter: #359962;
  --ifm-color-primary-lightest: #3cad6e;
  --ifm-code-font-size: 95%;
  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.1);
}

/* For readability concerns, you should choose a lighter palette in dark mode. */
[data-theme='dark'] {
  --ifm-color-primary: #25c2a0;
  --ifm-color-primary-dark: #21af90;
  --ifm-color-primary-darker: #1fa588;
  --ifm-color-primary-darkest: #1a8870;
  --ifm-color-primary-light: #29d5b0;
  --ifm-color-primary-lighter: #32d8b4;
  --ifm-color-primary-lightest: #4fddbf;
  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.3);
}

/* Custom styles for robotics content */
.robotics-diagram {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  margin: 16px 0;
  background-color: #f9f9f9;
}

.robotics-diagram img {
  max-width: 100%;
  height: auto;
}

.code-block-title {
  font-size: 0.85em;
  color: #666;
  margin-bottom: 0.5em;
  display: block;
}

/* Enhanced table styling */
.markdown table {
  display: table;
  width: 100%;
}

.markdown table th,
.markdown table td {
  border: 1px solid var(--ifm-color-emphasis-300);
  padding: 0.75rem;
}

.markdown table th {
  background-color: var(--ifm-color-emphasis-100);
  font-weight: bold;
}

/* Custom admonitions for academic content */
.admonition-implementation {
  border-left-color: #42b983;
}

.admonition-caution {
  border-left-color: #ff6b6b;
}

.admonition-note {
  border-left-color: #4a90e2;
}
```

## 4. Book Content Structure

### 4.1 docs/intro.md

```markdown
---
sidebar_position: 1
---

# Introduction to Physical AI & Humanoid Robotics

## Overview

Welcome to the comprehensive academic book on **Physical AI & Humanoid Robotics**. This book provides in-depth coverage of modern robotics systems with a focus on Physical AI, covering essential topics from robotic operating systems to advanced AI integration.

## Target Audience

This book is designed for:
- Computer science students studying robotics
- Researchers in AI and robotics
- Engineers working on robotic systems
- Anyone interested in the intersection of AI and physical systems

## Book Structure

The book is organized into four main modules:

1. **ROS 2 - The Robotic Nervous System**: Core concepts of Robot Operating System 2
2. **Gazebo/Unity - Digital Twin**: Simulation environments and digital twin technology
3. **NVIDIA Isaac - The AI Robot Brain**: AI integration and GPU-accelerated robotics
4. **VLA & Capstone Project**: Vision-Language-Action models and comprehensive implementation

## Learning Objectives

By the end of this book, readers will be able to:
- Understand and implement ROS 2-based robotic systems
- Create and utilize digital twin environments for robotics
- Integrate AI capabilities using NVIDIA Isaac platform
- Develop Vision-Language-Action models for robotic applications
- Complete a comprehensive capstone project integrating all concepts

## Prerequisites

- Basic understanding of programming concepts
- Familiarity with Linux command line
- Basic knowledge of mathematics (linear algebra, calculus)
- Understanding of basic robotics concepts (helpful but not required)

## How to Use This Book

This book is designed to be both a learning resource and a reference. Each chapter includes:
- Theoretical concepts with practical examples
- Code snippets and implementation details
- Exercises and projects for hands-on learning
- References to academic papers and resources
```

### 4.2 docs/ros2/index.md

```markdown
---
sidebar_position: 2
---

# ROS 2 - The Robotic Nervous System

## Chapter Overview

This chapter covers the Robot Operating System 2 (ROS 2), which serves as the central nervous system for modern robotics applications. We'll explore the architecture, communication patterns, and practical implementation techniques essential for developing sophisticated humanoid robotics systems.

## Learning Objectives

By the end of this chapter, readers will be able to:
- Understand the fundamental architecture of ROS 2
- Implement nodes, topics, services, and actions
- Configure Quality of Service (QoS) policies for different robotic applications
- Design effective communication patterns for complex robotic systems
- Debug and monitor ROS 2 systems efficiently

## Table of Contents

- [Core Concepts](./concepts.md)
- [Node Communication](./communication.md)
- [Quality of Service](./qos.md)
- [Practical Examples](./examples.md)
```

### 4.3 docs/gazebo-unity/index.md

```markdown
---
sidebar_position: 3
---

# Gazebo/Unity - Digital Twin Environments

## Chapter Overview

Digital twin technology plays a crucial role in robotics development, allowing for safe, cost-effective testing and validation of complex robotic systems before deployment in the real world. This chapter explores two leading simulation environments—Gazebo and Unity—and their applications in robotics development.

## Learning Objectives

By the end of this chapter, readers will be able to:
- Understand the principles and benefits of digital twin technology in robotics
- Set up and configure Gazebo and Unity simulation environments
- Create realistic robot models and environments for simulation
- Integrate simulation with ROS 2 for hardware-in-the-loop testing
- Implement physics-based sensor simulation and realistic perception

## Table of Contents

- [Digital Twin Concepts](./concepts.md)
- [Gazebo Simulation](./gazebo.md)
- [Unity Integration](./unity.md)
- [Practical Applications](./applications.md)
```

### 4.4 docs/nvidia-isaac/index.md

```markdown
---
sidebar_position: 4
---

# NVIDIA Isaac - The AI Robot Brain

## Chapter Overview

NVIDIA Isaac represents a comprehensive platform for developing AI-powered robotics applications, combining hardware acceleration, software frameworks, and development tools into an integrated solution. This chapter explores the NVIDIA Isaac ecosystem and its applications in robotics.

## Learning Objectives

By the end of this chapter, readers will be able to:
- Understand the NVIDIA Isaac platform architecture and components
- Implement GPU-accelerated perception pipelines using Isaac ROS
- Utilize Isaac Sim for advanced robotics simulation and training
- Deploy AI models for robotics applications using Isaac frameworks
- Integrate Isaac capabilities with ROS 2 for hybrid robotic systems

## Table of Contents

- [Isaac Platform Overview](./overview.md)
- [Isaac ROS Integration](./ros.md)
- [Isaac Sim Applications](./sim.md)
- [AI Model Deployment](./deployment.md)
```

### 4.5 docs/vla-capstone/index.md

```markdown
---
sidebar_position: 5
---

# Vision-Language-Action Models and Capstone Project

## Chapter Overview

Vision-Language-Action (VLA) models represent a paradigm shift in robotics, enabling robots to understand natural language commands and execute complex tasks in visual environments. This chapter explores VLA models and concludes with a comprehensive capstone project.

## Learning Objectives

By the end of this chapter, readers will be able to:
- Understand the architecture and principles of Vision-Language-Action models
- Implement voice-to-action pipelines for humanoid robots
- Integrate VLA models with ROS 2 and NVIDIA Isaac systems
- Design and execute complex robotic tasks using natural language commands
- Build a complete humanoid robot system that responds to voice commands

## Table of Contents

- [VLA Model Architecture](./architecture.md)
- [Voice-to-Action Pipeline](./voice-action.md)
- [Integration Patterns](./integration.md)
- [Capstone Project](./capstone.md)
```

## 5. Custom Components

### 5.1 Create Custom Components Directory

```bash
mkdir -p src/components
```

### 5.2 Interactive Diagram Component (src/components/InteractiveDiagram.js)

```javascript
// src/components/InteractiveDiagram.js
import React from 'react';
import clsx from 'clsx';
import styles from './InteractiveDiagram.module.css';

const FeatureList = [
  {
    title: 'ROS 2 Architecture',
    description: (
      <>
        The Robot Operating System 2 provides the communication backbone for robotic systems.
      </>
    ),
  },
  {
    title: 'Digital Twin',
    description: (
      <>
        Simulation environments that mirror real-world robotic systems for safe testing.
      </>
    ),
  },
  {
    title: 'AI Integration',
    description: (
      <>
        GPU-accelerated AI processing for perception, planning, and control.
      </>
    ),
  },
];

function Feature({ Svg, title, description }) {
  return (
    <div className={clsx('col col--4')}>
      <div className="text--center">
        {/* Add your SVG here */}
      </div>
      <div className="text--center padding-horiz--md">
        <h3>{title}</h3>
        <p>{description}</p>
      </div>
    </div>
  );
}

export default function InteractiveDiagram() {
  return (
    <section className={styles.features}>
      <div className="container">
        <div className="row">
          {FeatureList.map((props, idx) => (
            <Feature key={idx} {...props} />
          ))}
        </div>
      </div>
    </section>
  );
}
```

### 5.3 Chatbot Integration Page

Create a chatbot integration page:

```markdown
<!-- src/pages/chatbot.md -->
# Book Chatbot

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## Interactive Book Assistant

This page provides access to the RAG (Retrieval-Augmented Generation) chatbot that can answer questions about the Physical AI & Humanoid Robotics book content.

<Tabs>
<TabItem value="embedded" label="Embedded Chatbot" default>

<div style={{height: '600px', border: '1px solid #ccc', borderRadius: '8px', overflow: 'hidden'}}>
  <iframe
    src="http://localhost:8000"
    width="100%"
    height="100%"
    title="Book Chatbot"
    style={{border: 'none'}}
  ></iframe>
</div>

</TabItem>
<TabItem value="api" label="API Access">

### API Endpoints

The chatbot API is available at:
- **Query Endpoint**: `POST /query`
- **Health Check**: `GET /health`

### Example Request

```bash
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is ROS 2?",
    "max_results": 5
  }'
```

</TabItem>
</Tabs>

## How It Works

The chatbot uses:
- **Vector Database**: Qdrant Cloud for semantic search
- **AI Model**: OpenAI GPT for answer generation
- **Content**: Book chapters and concepts
- **Integration**: Real-time question answering
```

## 6. Package.json Configuration

Update the package.json file:

```json
{
  "name": "physical-ai-humanoid-robotics-book",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "docusaurus": "docusaurus",
    "start": "docusaurus start",
    "build": "docusaurus build",
    "swizzle": "docusaurus swizzle",
    "deploy": "docusaurus deploy",
    "clear": "docusaurus clear",
    "serve": "docusaurus serve",
    "write-translations": "docusaurus write-translations",
    "write-heading-ids": "docusaurus write-heading-ids"
  },
  "dependencies": {
    "@docusaurus/core": "3.0.0",
    "@docusaurus/preset-classic": "3.0.0",
    "@mdx-js/react": "^3.0.0",
    "clsx": "^2.0.0",
    "prism-react-renderer": "^2.3.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@docusaurus/module-type-aliases": "3.0.0",
    "@docusaurus/types": "3.0.0"
  },
  "engines": {
    "node": ">=18.0"
  },
  "browserslist": {
    "production": [
      ">0.5%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 3 chrome version",
      "last 3 firefox version",
      "last 5 safari version"
    ]
  },
  "engines": {
    "node": ">=18.0"
  }
}
```

## 7. GitHub Actions for Automated Deployment

Create a GitHub Actions workflow for automated deployment:

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    name: Deploy to GitHub Pages
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: npm

      - name: Install dependencies
        run: npm install

      - name: Build website
        run: npm run build

      # Popular action to deploy to GitHub Pages:
      # Docs: https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-docusaurus
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          # Build output to publish to the `gh-pages` branch:
          publish_dir: ./build
          # The following lines assign commit authorship to the official
          # GH-Actions bot for deploys to `gh-pages` branch:
          # https://github.com/actions/checkout/issues/13#issuecomment-724415212
          # The GH actions bot is used by default if you didn't specify the two fields.
          # You can swap them out with your own user credentials.
          user_name: github-actions[bot]
          user_email: 41898282+github-actions[bot]@users.noreply.github.com
```

## 8. Additional Configuration Files

### 8.1 .gitignore

```gitignore
# Dependencies
node_modules/

# Production builds
build/

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Docusaurus cache
.docusaurus/

# Temporary files
.temp/
.tmp/

# OS generated files
.DS_Store
Thumbs.db

# Logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Coverage directory used by tools like istanbul
coverage/

# Output of `npx @docusaurus/module-type-aliases`
docusaurus.config.js.swp
```

### 8.2 Static Files Setup

Create the static directory structure:

```bash
mkdir -p static/img
# Add your logo, diagrams, and other static assets to this directory
```

## 9. Deployment Instructions

### 9.1 Local Development

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Start development server**:
   ```bash
   npm start
   ```

3. **Access the site** at `http://localhost:3000`

### 9.2 Production Build

1. **Build the site**:
   ```bash
   npm run build
   ```

2. **Serve locally for testing**:
   ```bash
   npm run serve
   ```

### 9.3 GitHub Pages Deployment

1. **Enable GitHub Pages** in your repository settings:
   - Go to Settings → Pages
   - Select source as "Deploy from a branch"
   - Choose "gh-pages" branch

2. **The site will be automatically deployed** when you push to the main branch

### 9.4 Custom Domain (Optional)

If you want to use a custom domain:

1. **Add CNAME file** in static directory:
   ```
   your-domain.com
   ```

2. **Update docusaurus.config.js**:
   ```javascript
   baseUrl: '/',
   url: 'https://your-domain.com',
   ```

## 10. SEO and Analytics Configuration

### 10.1 Add SEO Metadata

Update `docusaurus.config.js` to include SEO metadata:

```javascript
// In docusaurus.config.js
module.exports = {
  // ... other config
  presets: [
    [
      'classic',
      {
        docs: {
          // ... docs config
          showLastUpdateTime: true,
          editCurrentVersion: true,
        },
        theme: {
          customCss: require.resolve('./src/css/custom.css'),
        },
      },
    ],
  ],
  themes: [
    [
      '@docusaurus/theme-classic',
      {
        customCss: require.resolve('./src/css/custom.css'),
      },
    ],
    [
      '@docusaurus/theme-search-algolia',
      {
        // The application ID provided by Algolia
        appId: 'YOUR_APP_ID',
        // Public API key
        apiKey: 'YOUR_SEARCH_API_KEY',
        indexName: 'your-index-name',
        // Optional: see doc section below
        contextualSearch: true,
        // Optional: Specify domains where the navigation should occur through window.location instead on history.push. Useful when our Algolia config crawls multiple documentation sites and we want to navigate with window.location.href to them.
        externalUrlRegex: 'external\\.com|domain\\.com',
        // Optional: Replace parts of the item URLs from Algolia. Useful when using the same search index for multiple deployments using a different baseUrl. You can use regexp or string in the `from` param. For example: localhost:3000 vs myCompany.com/docs
        replaceSearchResultPathname: {
          from: '/docs/', // or as RegExp: /\/docs\//
          to: '/',
        },
        // Optional: Algolia search parameters
        searchParameters: {},
        // Optional: path for search page that enabled by default (`false` to disable it)
        searchPagePath: 'search',
      },
    ],
  ],
};
```

## 11. Content Migration

To migrate your existing book content to this Docusaurus structure:

1. **Organize your MDX files** according to the sidebar structure
2. **Update frontmatter** in each file with proper metadata
3. **Create navigation links** in sidebars.js
4. **Test the site** locally before deployment

## 12. Advanced Features

### 12.1 Adding Search

Docusaurus comes with Algolia search built-in. To enable it, you'll need to sign up for Algolia and get your API keys.

### 12.2 Adding Analytics

```javascript
// In docusaurus.config.js
module.exports = {
  // ... other config
  scripts: [
    {
      src: 'https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID',
      async: true,
    },
  ],
  plugins: [
    [
      '@docusaurus/plugin-google-gtag',
      {
        trackingID: 'GA_MEASUREMENT_ID',
        anonymizeIP: true,
      },
    ],
  ],
};
```

This Docusaurus/Spec-Kit Plus deployment setup provides:

1. **Complete project structure** for the book
2. **Proper configuration files** for Docusaurus
3. **GitHub Actions workflow** for automated deployment
4. **Custom components** for enhanced user experience
5. **SEO and analytics configuration** for professional deployment
6. **Detailed deployment instructions** for GitHub Pages
7. **Content organization guidelines** for easy maintenance

The setup is designed to be scalable, maintainable, and production-ready, providing an excellent reading experience for the Physical AI & Humanoid Robotics book.