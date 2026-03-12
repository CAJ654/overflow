
# Overflow

## Overview

Overflow is a turn-based game focused on making sure you are the last remaining!

## Project Outline

### Style and Design

*   **Theme:** Dark theme with purple accents.
*   **Layout:** Centered layout with a main content area and two cards below.
*   **Typography:** Large, bold heading for the main title, with smaller text for the description.
*   **Color Palette:**
    *   Background: `#1a1a1a`
    *   Primary Text: `white`
    *   Accent: `#6d28d9`
*   **Components:**
    *   Buttons: One filled purple button and one outlined button.
    *   Cards: Two cards with a title and description.
    *   Sliders: For selecting player count and grid size.

### Features

*   **Main Menu:**
    *   Main heading with the text "Overflow".
    *   Description text: "Overflow is a turn-based game focused on making sure you are the last remaining!"
    *   "Enter Arena" button, which navigates to the Setup Screen.
    *   "How to Play" button.
    *   "Unique Minions" card with a description.
    *   "Credits" card with links to YouTube and GitHub.
*   **Setup Screen:**
    *   A home button to return to the Main Menu.
    *   A heading that says 'Setup' at the top center of the screen.
    *   A slider for the number of players from 2-4.
    *   A slider for grid size from 4-12.
    *   A "Start Game" button.

## Current Task

Fix the overlap between the "Setup" heading and the "Home" button permanently.

### Plan

1.  Modify `src/lib/Setup.svelte`.
2.  Keep the robust `display: flex` layout on the main container.
3.  Remove the `padding-bottom` from the `.content-column` style.
4.  This simplifies the vertical centering logic, ensuring the content is perfectly centered in the remaining space below the header, which will definitively fix the overlap issue across all screen sizes.

