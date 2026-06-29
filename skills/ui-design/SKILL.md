---
name: UI Design
description: >
    UI design guidelines including Light/Dark mode support, custom theme transitions,
    and premium interactive reveal animations. Apply this skill when designing, building,
    or updating web interfaces, widgets, dashboards, or single-page apps.
---

# UI Design & Theme Transitions

These rules specify the standard UI architecture, styling conventions, and animations required for web interfaces, in particular Light/Dark mode transitions.

---

## Theme Toggle & Theme Switch Architecture

Every web application and dashboard must support both light and dark themes unless explicitly requested otherwise. The theme system must follow this architecture:

### 1. Root Variable Definition

Define standard theme colors as CSS custom properties on `:root` (representing the default dark theme) and override them under `html.light-theme` (for the light theme).

- **Dark Mode (Default)**: Use vibrant, glowing accent colors against deep dark-grey/black backgrounds.
- **Light Mode**: Use clean, high-contrast, soft off-white and light-grey backgrounds with readable, slightly darker/saturated versions of colors.

### 2. Instant Theme Loader

To prevent any flash of dark mode (FOUC - Flash of Unstyled Content) on reload, place a blocking script at the very top of `<head>`:

```html
<script>
    (function () {
        const savedTheme = localStorage.getItem("theme");
        if (savedTheme === "light") {
            document.documentElement.classList.add("light-theme");
        }
    })();
</script>
```

### 3. Floating / Integrated Theme Toggle Button

Provide an accessible and beautifully animated theme toggle button:

```html
<button
    id="theme-toggle"
    class="theme-toggle-btn"
    aria-label="Toggle theme"
    onclick="toggleTheme(event)"
>
    <span class="sun-icon">☀️</span>
    <span class="moon-icon">🌙</span>
</button>
```

---

## Theme Transitions & Reveal Animations

### 1. Circular Reveal Animation (View Transitions API)

When toggling theme, use the View Transitions API to perform a modern radial reveal sweeping outward from the user's cursor position.

```javascript
function toggleTheme(event) {
    const isTransitionable =
        document.startViewTransition &&
        !window.matchMedia("(prefers-reduced-motion: reduce)").matches;

    if (!isTransitionable) {
        const isLight =
            document.documentElement.classList.toggle("light-theme");
        localStorage.setItem("theme", isLight ? "light" : "dark");
        return;
    }

    // Use cursor coordinates or fall back to center
    let x = window.innerWidth / 2;
    let y = window.innerHeight / 2;
    if (event) {
        x = event.clientX;
        y = event.clientY;
    }

    const endRadius = Math.hypot(
        Math.max(x, window.innerWidth - x),
        Math.max(y, window.innerHeight - y),
    );

    const transition = document.startViewTransition(() => {
        const isLight =
            document.documentElement.classList.toggle("light-theme");
        localStorage.setItem("theme", isLight ? "light" : "dark");
    });

    transition.ready.then(() => {
        document.documentElement.animate(
            {
                clipPath: [
                    `circle(0px at ${x}px ${y}px)`,
                    `circle(${endRadius}px at ${x}px ${y}px)`,
                ],
            },
            {
                duration: 650,
                easing: "cubic-bezier(0.4, 0, 0.2, 1)",
                pseudoElement: "::view-transition-new(root)",
            },
        );
    });
}
```

### 2. Guarding Classical Page Transitions

Disable standard CSS cross-fade transitions during the View Transition to prevent stuttering. Use the `@supports` query to apply fade-out transitions only to browsers that don't support View Transitions:

```css
/* Only apply standard transitions if View Transitions are not supported */
@supports not (view-transition-name: root) {
    body,
    .card,
    .table,
    .button {
        transition:
            background-color 0.4s ease,
            border-color 0.4s ease,
            color 0.4s ease;
    }
}

/* Override browser default view transitions cross-fade */
::view-transition-old(root),
::view-transition-new(root) {
    animation: none;
    mix-blend-mode: normal;
}
::view-transition-new(root) {
    z-index: 9999;
}
::view-transition-old(root) {
    z-index: 1;
}
```

### 3. Icon Rotate & Scale Animation

Animate toggle buttons using scaling and rotation transitions for sun/moon icons:

```css
.theme-toggle-btn {
    position: relative;
    overflow: hidden;
}
.theme-toggle-btn span {
    position: absolute;
    transition:
        transform 0.5s cubic-bezier(0.4, 0, 0.2, 1),
        opacity 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}
.sun-icon {
    opacity: 0;
    transform: rotate(90deg) scale(0);
}
.moon-icon {
    opacity: 1;
    transform: rotate(0) scale(1);
}
.light-theme .sun-icon {
    opacity: 1;
    transform: rotate(0) scale(1);
}
.light-theme .moon-icon {
    opacity: 0;
    transform: rotate(-90deg) scale(0);
}
```
