# QUITR - Cold Turkey Addiction Recovery App

## Project Overview
- **Project Name**: QUITR
- **Type**: Single-page webapp (HTML/Tailwind CSS)
- **Core Functionality**: A motivational tool for quitting AI sexbot addiction cold turkey, featuring a counter, affirmations, emergency distractions, and progress visualization
- **Target Users**: People committed to breaking AI sexbot addiction

## UI/UX Specification

### Layout Structure
- Single page, vertically centered content
- Hero section with app title and tagline
- Main counter display (hours/minutes sober)
- Action buttons for emergency and check-ins
- Motivational cards section
- Footer with resources

### Visual Design

**Color Palette**
- Background: Deep charcoal `#0D0D0D` with subtle noise texture
- Primary: Electric cyan `#00F5D4` (sober energy)
- Secondary: Warm amber `#FF9F1C` (urgency/warning)
- Accent: Soft rose `#F72585` ( temptation alert)
- Text primary: Off-white `#F8F8F2`
- Text muted: `#6B7280`
- Card backgrounds: `#1A1A2E` with subtle borders

**Typography**
- Headings: "Syne" (bold, distinctive) - from Google Fonts
- Body: "DM Sans" - from Google Fonts
- Counter display: Monospace variant for numbers

**Spacing**
- Generous padding (2rem+ sections)
- Card padding: 1.5rem
- Button padding: 1rem 2rem

**Visual Effects**
- Counter has a pulsing glow animation
- Cards have glass-morphism effect with subtle blur
- Buttons have hover scale and glow effects
- Background has animated gradient orbs (subtle)
- Page load: staggered fade-in animation

### Components

**1. Hero Section**
- App name "QUITR" in large bold text
- Tagline: "Cold Turkey. Cold Truth. New You."
- Subtitle explaining commitment

**2. Sobriety Counter**
- Large digital display showing hours:minutes:seconds
- Starts at 00:00:00 on "I commit" click
- Pulsing cyan glow behind numbers
- States: "Not started" → "Sobering..." → "Sober"

**3. I Commit Button**
- Large prominent button
- Amber/orange gradient
- Click triggers counter start
- Has shake animation on hover

**4. Emergency Distraction Section**
- Grid of 4 distraction cards
- Each card has icon + quick action
- Activities: Call friend, Exercise, Journal, Meditation

**5. Motivational Quotes**
- Rotating quote display
- Shows one quote at a time from curated list
- Auto-rotates every 10 seconds

**6. Progress Visualization**
- Simple visual bar showing hours reached
- Milestones marked (1hr, 24hr, 72hr, 1 week)

## Functionality Specification

### Core Features
1. **Counter**: Starts on commit, counts up in real-time
2. **LocalStorage**: Persists counter across page refreshes
3. **Distraction quick-links**: Placeholder buttons for common distractions
4. **Quote rotation**: Cycling through motivational quotes
5. **Milestone tracking**: Visual progress toward key goals

### User Interactions
- Click "I Commit" to start journey
- Counter persists until manually reset
- Click distraction cards for simple feedback
- Click quote to manually advance

### Data Handling
- Start timestamp stored in localStorage
- Counter calculates difference on each render

## Acceptance Criteria
- [ ] Page loads with all visual elements visible
- [ ] Counter starts at 00:00:00 and counts up properly
- [ ] Counter persists across page refresh
- [ ] Animations are smooth and performant
- [ ] All text is readable on dark background
- [ ] Mobile responsive (looks good on phone)