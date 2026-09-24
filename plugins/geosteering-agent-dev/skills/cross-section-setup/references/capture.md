# Capturing the finished scene

When the user wants the result as an image file (saved, cropped, or handed
to another workflow):

1. **Tidy first.** Close every pane and dialog; turn off cursor info and
   legend (Scene pane) unless wanted; make sure the loading overlay,
   tooltips, and the floating edit bar aren't mid-interaction. Take a final
   full screenshot and check it — this is the frame you'll crop.
2. **Capture.** Take the screenshot with `save_to_disk: true` (the
   `computer`/screenshot action). Note the saved path from the tool result.
3. **Crop.** Use Python + PIL in the sandbox. Don't hardcode pixels —
   measure from the screenshot you just took. The usual crop keeps the
   section plus its depth axis and the two log tracks, and removes the app
   tab bar and the pane menu strip (everything above the active log track).
   If the user wants "just the section", also crop off the type log track
   at the left and the active log strip at the top.
4. **Deliver.** Save the cropped PNG to the outputs folder with a
   descriptive name (`<project>-cross-section.png`) and present it to the
   user. If they asked for email/slides/docs, hand the file off to the
   appropriate tool or skill — that's beyond this skill's scope.

The floating edit bar at the bottom center of the canvas will appear in
captures; it's small and centered at the bottom, so a crop that trims the
bottom ~40 px of the canvas removes it if the user wants a perfectly clean
image.
