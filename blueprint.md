
# Overflow Game Blueprint

## Overview

This document outlines the blueprint for a web-based game called "Overflow". The game is a turn-based strategy game with a simple premise: be the last player standing. This document details the project's features, design, and implementation.

## Features

- **Two Game Modes:**
  - **Overflow:** The last player remaining on the board wins.
  - **Underflow:** The first player to be eliminated from the board wins.
- **Customizable Game Setup:**
  - **Player Count:** 2 to 4 players.
  - **Grid Size:** 4x4 to 12x12.
  - **Player Colors:** Players can choose their own colors.
- **Light and Dark Modes:** The application features a dynamic theme that can be toggled between light and dark modes.

## Design

- **Color Palette:**
  - **Dark Mode:**
    - Background: `#1a1a1a`
    - Text: `white`
    - Primary Accent: `#6d28d9`
    - Card Background: `#2a2a2a`
  - **Light Mode:**
    - Background: `#f0f0f0`
    - Text: `#1a1a1a`
    - Primary Accent: `#92d726`
    - Card Background: `#d5d5d5`
- **Typography:** A clean, sans-serif font is used for readability.

## Implementation Plan: Theming

The following steps have been taken to implement the light and dark mode functionality:

1.  **Global Theme Management:**
    - A global stylesheet in `App.svelte` defines CSS variables for the color palette.
    - A `light-mode` class is toggled on the root element to switch between themes.
2.  **Theme Toggle Button:**
    - A button in the top-right corner of the screen toggles the `light-mode` class on the root element.
    - The button's icon changes to reflect the current theme (sun for light, moon for dark).
3.  **Component Styling:**
    - All components have been updated to use the CSS variables defined in the global stylesheet.
    - This ensures that all components will automatically update their styles when the theme changes.
