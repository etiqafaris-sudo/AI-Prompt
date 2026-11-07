# **Recreate hero section**

Recreate this exact hero section for my AI image generator app, keeping the same overall layout, spacing, typography hierarchy, oversized brand text, and premium editorial feel. Replace the existing person and portfolio content with my AI image-generation product. Use a dark futuristic background with subtle purple/violet accents. Replace the large “Dominic” text with my brand name DREAMFRAME. Add a small availability-style badge such as AI IMAGE GENERATOR, a short headline about creating images with AI, a concise supporting description, and an orange/purple CTA button like Start Creating. Keep the design minimal, bold, modern, and highly polished. Make the hero responsive on mobile. Next.js is already installed and configured in the project, so build directly within the existing Next.js setup. Do not install or replace the framework.

# **Scroll Animation Only**

I already built the hero section and UI. Do **not** change the layout, text, navbar, buttons, spacing, colors, or styling.

Create **only** a cinematic **scroll-linked image sequence animation** behind the existing hero using the image frames already in the project folder.

## **CORE INTERACTION**

Build a full-screen **sticky HTML5 canvas** that stays pinned while the user scrolls.

The animation must be driven by **scroll progress**, not by time.

**Use the images inside Zip file in the project folder: public\frames_150.zip**

**frame-001.png**

through

**frame-150.png**

The canvas must:

- preload frames progressively
- render the correct frame based on scroll position
- update continuously while scrolling
- never get stuck on the first frame
- use requestAnimationFrame
- use smooth interpolation/lerp between frames
- support retina/DPR screens
- stay fully responsive
- avoid flicker, jumps, or layout shift
- respect prefers-reduced-motion

## **SCROLL LOGIC**

The hero section should be a tall scroll area, around h-[400vh]or more.

Map scroll progress directly to frame index:

- 0% scroll → frame 1
- 100% scroll → frame 150

Important:

- The frame must recalculate on every scroll change
- The canvas must redraw whenever the target frame changes
- Do not rely on a one-time initial render
- Do not stop updating after the first frame
- Use a persistent animation loop so the current frame eases toward the target frame smoothly

## **VISUAL DIRECTION**

The animation should feel:

- luxurious
- cinematic
- futuristic
- minimal
- ultra-premium
- Apple-level smooth

Background:

- deep gold / dark luxury tone
- seamless canvas and background color
- no visible edges
- no distracting effects

# **Refine the hero**

**Refine the existing DreamFrame hero only. Keep the current background, layout, navigation, badge, CTA, and all existing content — do not add or remove any elements.**

Make the left headline significantly more readable by adding a **large, smooth cinematic dark shadow/gradient behind the text**, strongest around the left 10–25% of the screen and gradually fading out toward 45–50%. It should have **no hard edges or visible box**, just a soft atmospheric shadow integrated into the background.

Make the headline slightly bolder with a clean **white → subtle lavender gradient** and a soft shadow/glow.

Make **DREAMFRAME** the main visual: centered horizontally, approximately **90vw wide**, positioned at the bottom, with large elegant typography, white/lavender gradient, subtle pink glow, and soft depth shadow.

Keep everything **minimal, spacious, and cinematic**. Do not introduce any new UI elements or redesign the background. The goal is simply to make the existing hero feel more premium and dramatically improve text contrast.

# **Build the next two sections**

**Build the next two sections directly below my existing hero without changing the hero.**

**1. Infinite Logo Carousel:** Create two seamless horizontal logo marquees. The top row moves continuously **right → left**, and the second row **left → right**. Use smooth infinite looping, staggered logo spacing, subtle edge fade/masking, and no visible jumps. Keep it minimal and premium.

**2. Bento Grid:** Create the section immediately after the carousels, using the **attached image as the exact structural/layout reference**. Recreate its asymmetric grid, card proportions, rounded corners, spacing, and visual hierarchy, but adapt the colors, typography, content, and imagery to my existing website design. **Do not copy the image's content—only its layout structure.**

**For both sections, use a dark cosmic background with glassy gradient elements, soft purple/pink/lavender glowing effects, and colors that match the existing DreamFrame hero.**

Make both sections fully responsive and visually consistent with the existing hero

# **3D Model Creator section**

Create a new 3D Model Creator section using the attached design as the exact layout reference. Adapt it to my AI image-generation platform, remove all center elements/background from the reference, and use a clean light-gray background for the section. Keep the same layout, proportions, and visual style.