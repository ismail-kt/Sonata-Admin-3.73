# Sonata Admin 3.x & 4.x Documentation

This repository now contains side-by-side documentation for Sonata Admin 3.73 and a suggested 4.x documentation set.

Docs:
- docs/3.73/ — reference material and examples targeted at Sonata Admin 3.73 (PHP 7.4, Symfony 4.4 era).
- docs/4.x/ — updated guidance, examples, and migration notes for Sonata Admin 4.x (PHP ^8.2, modern Symfony).

You can keep both sets of docs while you incrementally migrate code. If you want, I can now: scan the repository for files and patterns that will need changes for a 4.x migration (composer.json, usages of `$this->container`, old menu helpers, typed signatures), and produce a prioritized report.
