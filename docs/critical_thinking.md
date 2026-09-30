# Critical Thinking — ICA 06: Drawing with Flutter
**Student:** Wilder Edwards  
**Date:** September 26, 2026

## Undergraduate: Measure and improve your smile arc

For my smile I sketched it out first: face center in the middle of the canvas, radius as a fraction of the canvas, then a mouth Rect below center with a start and sweep angle for `drawArc`. In code that ended up as:

- `center (c) = Offset(size.width / 2, size.height / 2)`
- `radius (r) = size.shortestSide * 0.40`
- `mouthRect = Rect.fromCenter(center: Offset(c.dx, c.dy + r * 0.15), width: r * 1.0, height: r * (0.4 + mood * 0.5))`
- happy (`mood >= 0.5`): `drawArc(mouthRect, 0.15 * pi, 0.70 * pi, false, mouthPaint)`
- sad (`mood < 0.5`): `frownRect = mouthRect.translate(0, r * 0.25)` then `drawArc(frownRect, 1.15 * pi, 0.70 * pi, false, mouthPaint)`
- eyes for reference: `eyeY = c.dy - r * 0.18`, `eyeDx = r * 0.35`

What I saw on the emulator was pretty clear. At `mood = 0.8` I get a centered yellow smile, at `~0.5` it gets smaller/flatter, and under `0.5` it flips to a frown and the face turns blue from my `determineFaceColor()` logic. The slider updates instantly because of `setState()` -> new `SmileyPainter(mood: mood)` -> `shouldRepaint == true`.

For responsiveness I tested portrait and then rotated to landscape. The main fix that made it work was switching everything mouth-related off fixed pixels and onto `r` fractions, and using `size.shortestSide` instead of just `size.width`. That way when the height gets smaller in landscape the face still fits and doesn't get clipped. Honestly my hat is still hard-coded (`Offset(c.dx, c.dy - 120)`, `100x20`) and my closed-eye rects are hard-coded too, so those are the next things I'd convert to `r * ...` like I did for the eyes.

`shouldRepaint` should return `true` when `mood` changes because otherwise Flutter would just keep the old frame and my mouth would never move. It should return `false` when nothing changed because `paint()` is expensive — there's no reason to redraw the exact same circles and arcs every build.

- [x] Smile arc is centered and uses `drawArc`
- [x] Mouth coordinates are based on `size` / `radius`
- [x] Tested portrait + landscape on phone emulator
- [x] I can explain the `shouldRepaint` decision

*Screenshot: see `docs/smiley_portrait.png`*

## Graduate: When should a painter repaint?

I compared `return true;` vs `return oldDelegate.mood != mood || oldDelegate.eyeSize != eyeSize;` with the same smiley and sliders. With `always true`, Flutter reruns `paint()` on every rebuild even if I didn't touch anything, which is wasteful. With the field check, it only repaints when I actually move the mood slider (0.0-1.0, controls mouth height/geometry + face color) or the eye-size slider (10-50, controls `eyeRadius`).

So I'd definitely use the field-comparing version. Right now my code only does `oldDelegate.mood != mood`, which means eye-size changes can get ignored — I need to add `eyeSize` (and face color if I make it a field) to that check so every real input triggers a repaint and nothing else does.

- [x] Both versions tested with same app state
- [x] Redraw difference explained accurately
- [x] Recommendation tied to my actual fields (`mood`, `eyeSize`)
