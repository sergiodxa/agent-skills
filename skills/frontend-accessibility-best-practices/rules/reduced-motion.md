---
title: Respect Reduced Motion Preference
impact: MEDIUM
tags: [accessibility, animation, reduced-motion]
---

# Respect Reduced Motion Preference

Honor the user's `prefers-reduced-motion` setting to avoid triggering vestibular disorders.

## Why

- Some users experience motion sickness, dizziness, or seizures from animations
- Users explicitly request reduced motion in system preferences
- Respecting this preference is a WCAG 2.1 requirement (2.3.3)

## CSS Approach

Use the `motion-reduce` Tailwind variant:

```tsx
// Animation only plays if user hasn't requested reduced motion
<div className="animate-bounce motion-reduce:animate-none">
  Bouncing content
</div>

// Reduce transition duration
<div className="transition-all duration-300 motion-reduce:duration-0">
  Transitioning content
</div>
```

Or use CSS media query directly:

```css
@media (prefers-reduced-motion: reduce) {
  .animated-element {
    animation: none;
    transition: none;
  }
}
```

## JavaScript Approach

Use the `usePrefersReducedMotion` hook:

```tsx
import { usePrefersReducedMotion } from "~/hooks/use-prefers-reduced-motion";

function AnimatedCounter({ value }) {
  let prefersReducedMotion = usePrefersReducedMotion();

  if (prefersReducedMotion) {
    // Show static value immediately
    return <span>{value}</span>;
  }

  // Show animated count-up
  return <CountUp target={value} duration={1000} />;
}
```

### Hook Usage

```tsx
import { usePrefersReducedMotion } from "~/hooks/use-prefers-reduced-motion";

function CarouselControls() {
  let prefersReducedMotion = usePrefersReducedMotion();

  // Disable auto-play for reduced motion users
  let [isAutoPlaying, setIsAutoPlaying] = useState(!prefersReducedMotion);

  return (
    <Carousel autoPlay={isAutoPlaying && !prefersReducedMotion}>
      {/* slides */}
    </Carousel>
  );
}
```

## What to Reduce

### Should Respect Reduced Motion

- Decorative animations (floating elements, parallax)
- Auto-playing carousels and slideshows
- Animated backgrounds
- Page transition animations
- Count-up number animations
- Infinite scrolling marquees

### Can Keep (Essential Motion)

- Loading spinners (but keep them simple)
- Progress indicators
- Button hover states (if subtle)
- Form validation feedback
- Focus ring transitions (keep very short)

## Bad

```tsx
// Bad - no reduced motion consideration
function Banner() {
  return (
    <div className="animate-pulse">
      <FloatingParticles />
      <ParallaxBackground />
    </div>
  );
}
```

## Good

```tsx
function Banner() {
  let prefersReducedMotion = usePrefersReducedMotion();

  return (
    <div className="motion-reduce:animate-none animate-pulse">
      {!prefersReducedMotion && <FloatingParticles />}
      <StaticBackground />
    </div>
  );
}
```

## Rules

1. Use `motion-reduce:` Tailwind variant for CSS animations
2. Use `usePrefersReducedMotion` hook for JS-controlled animations
3. Disable auto-playing content for reduced motion users
4. Keep essential motion (loading indicators) but simplify them
5. Never use animations that flash or strobe rapidly
6. Provide static alternatives for decorative animations
