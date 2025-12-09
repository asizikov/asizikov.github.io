---
title: "Monitoring GitHub Copilot Adoption with Metrics"
date: 2025-12-09T00:00:00Z
slug: "copilot-metrics"
tags: ["github copilot", "metrics", "developer-tools"]
draft: false
---

## Introduction

In October 2025, GitHub announced the [Copilot usage metrics dashboard and API in public preview](https://github.blog/changelog/2025-10-28-copilot-usage-metrics-dashboard-and-api-in-public-preview/). This is a significant milestone for enterprise administrators who need to understand Copilot's impact and adoption within their organizations.

The question is no longer "are teams using AI?" but rather "how well are they using it?" and "what is the impact?"

## The State of Copilot Metrics

GitHub's native metrics dashboard provides valuable aggregated data at the enterprise level. It shows:

- Daily and weekly active users across GitHub Copilot IDE modes, including agent mode
- Agent adoption across your user base
- Lines of code added and deleted across all IDE modes
- Language and model usage patterns across IDEs

That said, it's still in Public Preview—the Product Team is actively adding new views. And administrators often need deeper insights at the user level to understand adoption patterns, identify champions, and provide targeted support to teams.

## Building a Custom Metrics Viewer (with Copilot, of course)

As someone supporting enterprises across EMEA with their Copilot adoption, I frequently need quick, user-level analysis of Copilot metrics. So I did what any developer would do—I vibe-coded my own solution using GitHub Copilot itself.

The **GitHub Copilot User-Level Metrics Viewer** is a client-side web application that transforms raw Copilot usage exports into actionable insights. You upload your metrics file, and it instantly visualizes everything from user engagement patterns to premium model utilization—all without your data ever leaving your browser.

The entire application was built end-to-end with GitHub Copilot, from the initial scaffolding to chart implementations and insights blocks. It was a fun experience and shows what's possible with AI-assisted development. 

## What the Viewer Offers

### Engagement & Adoption Analytics

The dashboard gives you a comprehensive overview of how your organization is adopting Copilot:

- **Daily Active Users**: Interactive charts showing the percentage of licensed users actively using Copilot each day.
- **Feature Adoption Funnel**: Visualize the progression from code completion → chat → agent mode, helping you understand the maturity of usage.
- **User Segmentation**: Quickly identify chat users, agent mode users, and completion-only users to target enablement efforts.

### Agent Mode & Chat Insights

Understanding how developers use different Copilot modes is crucial:

- **Chat Users by Mode**: Separate tracking for Ask mode, Edit mode, Agent mode, and Inline mode.
- **Daily Chat Requests**: Volume analysis across different interaction types.
- **Agent Mode Heatmap**: Visualize intensity of agent mode usage across days, including estimated service value calculations.

### Copilot Impact Analysis

One of the most requested features—measuring actual productivity impact:

- **Lines of Code Impact**: Track LOC added and deleted through different Copilot features.
- **Mode-by-Mode Breakdown**: Separate impact analysis for Agent Mode, Code Completion, Edit Mode, Inline Mode, and Ask Mode.
- **Combined Impact View**: Aggregate view of all Copilot contributions to your codebase.

### Premium Request Unit (PRU) Analysis

With premium models in Copilot, understanding consumption patterns is essential:

- **PRU vs Standard Model Usage**: Daily breakdown of requests by model type.
- **Model Feature Distribution**: See which features consume premium models.
- **Service Value Estimation**: Calculate estimated value based on multipliers.
- **Model-Specific Details**: Drill down into usage for GPT-4, Claude, and others.

### Plugin Version Tracking

A unique feature not available in native dashboards:

- **JetBrains & VS Code Extension Analysis**: Track which plugin versions your developers are using.
- **Outdated Plugin Detection**: Identify users with outdated extensions that may be missing features or have incomplete telemetry.
- **Upgrade Recommendations**: Actionable insights to drive plugin updates across your organization.

### Individual User Deep Dives

Sometimes you need granular visibility:

- **User Activity Calendars**: GitHub-style contribution calendars showing daily Copilot activity.
- **Per-User Statistics**: Interactions, code generation events, acceptance rates, and feature usage.
- **Language & IDE Preferences**: Understand how individual developers use Copilot across their workflow.

### Flexible Filtering

- **Date Range Filters**: Quick filters for last 7, 14, or 28 days, or view all data.
- **Language Filtering**: Option to exclude "unknown" language entries for cleaner analytics.
- **Real-Time Updates**: All visualizations update instantly as you apply filters.

## How It Works

1. **Export your data**: Use the GitHub API or dashboard to export user-level Copilot metrics (NDJSON or JSON format).
2. **Upload to the viewer**: Drag and drop or select your metrics file.
3. **Explore insights**: Navigate through Overview, Users, Languages, IDEs, Impact, PRU Usage, and Adoption views.

All processing happens client-side. Your data never leaves your browser, making it safe for enterprise use without additional security reviews.

## Why Build Custom Dashboards?

While GitHub's native dashboards are improving rapidly, custom tools fill some gaps:

- **Rapid Assessment**: Quickly identify power users and those who might need additional support.
- **Pattern Recognition**: Spot trends in language usage, IDE preferences, and feature adoption.
- **Privacy Control**: Client-side processing means complete control over sensitive user-level data.
- **Iteration Speed**: When you need a new visualization, you can ship it the same day.
- **Inspiration**: Use the tool as a reference for building your own integrations.

## The Path Forward: Integrated Analytics

This custom viewer is designed as a lightweight solution for immediate needs. For long-term analysis, I strongly recommend integrating Copilot metrics into your existing data analytics infrastructure. Tools like Power BI, Looker, Tableau, or similar platforms provide:

- Historical trend analysis
- Integration with other business metrics
- Custom dashboards tailored to your organization
- Advanced data visualization capabilities
- Automated reporting and alerts

The custom viewer can serve as inspiration for the types of metrics and visualizations most relevant to your organization's needs.

## Try It Out

The tool is open source and deployed to GitHub Pages. You can try it immediately at:

**[https://asizikov-demos.github.io/copilot-user-level-statistics-viewer/](https://asizikov-demos.github.io/copilot-user-level-statistics-viewer/)**

Simply upload your metrics file and start exploring. The source code is available on [GitHub](https://github.com/asizikov-demos/copilot-user-level-statistics-viewer).

## Conclusion

GitHub Copilot metrics are essential for understanding and improving AI adoption in your development organization. While GitHub's native dashboard provides a solid foundation, complementary tools can offer additional perspectives and insights.

Whether you use this custom viewer, build your own solution, or integrate directly with enterprise analytics platforms, the key is to move beyond basic adoption numbers and understand *how* your teams are using Copilot to transform their development workflows.

The AI era of software development is here—let's make sure we're measuring what matters.

---

> *I'm employed by GitHub at the time of writing this post. All opinions are my own.*
