---
layout: post
title: "Energy Monitor: A DIY Smart Grid Consumption Tracking System"
date: 2023-07-16
categories: [iot, diy, home-automation, energy-monitoring]
tags: [rust, raspberry-pi, energy-monitoring, influxdb, oled-display, rpict, linky, teleinfo]
excerpt: "Building a comprehensive energy monitoring system with Raspberry Pi, OLED display, and InfluxDB integration for tracking electrical consumption."
---

I'm excited to share another DIY project: an **Energy Monitor** that I built to track and analyze my home's electrical consumption. This comprehensive system measures grid consumption metrics, displays real-time data on an OLED screen, and stores everything in an InfluxDB database for long-term analysis.

<p align="center"><img height="500" alt="Energy Monitor Module" src="{{ '/assets/images/energymonitor.png' | relative_url }}" /></p>

The project was born from the principle that "You can't improve what you don't measure." I wanted to observe and store my energy consumption data to eventually optimize it, and well... it also looked like a really cool DIY project (which it turned out to be!).

The module was designed to fit any European-standard distribution board with the same form factor as a circuit breaker, featuring a 90mm (5-module) width. It doesn't collect data directly but rather fetches metrics from Lechacal's RPICT module and Enedis Linky electric meter (France's national power provider).

This project demonstrates how modern DIY electronics can create professional-grade energy monitoring solutions. The combination of Raspberry Pi, custom Rust application, and smart data visualization showcases the full spectrum of skills needed for comprehensive IoT projects. The Energy Monitor is now successfully tracking my electrical consumption and providing valuable insights through data visualization and analysis.

## Open Source & Documentation

All project files are available on GitHub:
- **Source Code**: [github.com/ncolomer/energy-monitor](https://github.com/ncolomer/energy-monitor)
- **Latest Release**: [![GitHub release (latest by date)](https://img.shields.io/github/v/release/ncolomer/energy-monitor)](https://github.com/ncolomer/energy-monitor/releases/latest)

The repository includes:
- Complete Rust application source code and compiled binaries for Raspberry Pi
- 3D printable enclosure designs (STL files)
- Comprehensive documentation and wiring diagrams
- Configuration examples and installation guides
- Hardware parts list with purchase links

---

*For detailed technical implementation, hardware setup, and configuration options, check out the complete project documentation on [GitHub](https://github.com/ncolomer/energy-monitor).*
