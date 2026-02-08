# Ionic Capacitor Skills

A collection of agent skills for building production-ready Ionic Capacitor mobile applications with Angular, React, or Vue.

## Installation

```bash
npx skills add erkamyaman/ionic-skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [ionic-capacitor](./skills/ionic-capacitor) | Full app scaffold with onboarding, paywall, ads, i18n, theming, and tabs — supports Angular, React, and Vue |

## What's Included

### ionic-capacitor

A comprehensive skill that guides AI agents to build complete Ionic Capacitor apps with:

| Feature | Implementation |
|---------|---------------|
| **Framework** | Ionic 8 + Angular 19 / React 19 / Vue 3.5 |
| **Native Runtime** | Capacitor 6 |
| **Onboarding** | Swipe screens with fullscreen video background + gradient overlay |
| **Paywall** | RevenueCat with weekly/yearly subscriptions + 50% OFF badge |
| **Ads** | AdMob banner ads via @capacitor-community/admob |
| **i18n** | Turkish/English — @ngx-translate (Angular), react-i18next (React), vue-i18n (Vue) |
| **Theming** | Light/Dark/System with CSS variables |
| **Navigation** | ion-tabs + framework-specific router |
| **Storage** | @capacitor/preferences |
| **Notifications** | @capacitor/push-notifications |
| **Auth Guard** | Onboarding route guard |
| **State** | Angular services / React hooks / Vue composables |

### App Flow

```
Onboarding (video bg) -> Paywall (subscriptions) -> Main App (tabs)
```

### Required Pages

- **Onboarding** - Fullscreen video background, swipe-based slides
- **Paywall** - Weekly/yearly subscription options, restore purchases
- **Settings** - Language, theme, notifications, remove ads, reset onboarding
- **Home** - Main content tab
- **Explore** - Discovery tab

## Usage

Skills activate automatically when agents detect relevant tasks:

- "Create a water reminder app" -> Sets up complete Ionic project with all features
- "Build a React mobile app" -> Sets up Ionic React project with all features
- "Build a Vue mobile app" -> Sets up Ionic Vue project with all features
- "Add onboarding to my app" -> Implements video background onboarding flow
- "Set up in-app purchases" -> Configures RevenueCat with paywall
- "Add ads to my Ionic app" -> Integrates AdMob with premium detection
- "Add dark mode" -> Implements theme switching service
- "Add Turkish language" -> Sets up i18n with TR/EN

## Tech Stack

- [Ionic Framework](https://ionicframework.com) - UI components
- [Angular](https://angular.dev) - Application framework
- [React](https://react.dev) - Application framework
- [Vue](https://vuejs.org) - Application framework
- [Capacitor](https://capacitorjs.com) - Native runtime
- [RevenueCat](https://www.revenuecat.com) - In-app purchases
- [@capacitor-community/admob](https://github.com/nicol-ograve/capacitor-community-admob) - Advertisements

## Architecture

The skill uses a **shared utilities + framework wrappers** pattern:

- **`utils/`** - Framework-agnostic TypeScript functions for Capacitor plugins (AdMob, RevenueCat, Preferences, etc.)
- **Angular** - `services/` wrapping utilities with `@Injectable`
- **React** - `hooks/` wrapping utilities with `useCallback`
- **Vue** - `composables/` wrapping utilities as composable functions

This minimizes duplication and keeps Capacitor logic in one place.

## Contributing

Add new skills by creating a folder in `skills/` with:
- `SKILL.md` - Instructions for agents
- `metadata.json` - Skill metadata (version, triggers, references)

## License

MIT
