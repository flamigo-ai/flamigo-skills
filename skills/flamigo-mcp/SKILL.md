---
name: flamigo-mcp
description: Turn rough notes into beautiful presentations with Flamigo, right from your AI assistant. Start with notes or an outline, refine individual slides, compare designs, and export to HTML or PowerPoint (Pro).
---

# Flamigo MCP

Paste your notes, review the outline, and let Flamigo design the slides. It pairs layouts, colors, type, and illustrations to suit your topic. Want a different look for one slide? Compare design previews and keep your favorite.

Flamigo saves presentations in the signed-in user's web workspace. A deck has three stages: an **outline** sets the point of each page, **slide DSL** stores structured content, and **rendered HTML** shows the design. Choose the entry point from the material the user already has and stop at the deliverable they requested.

## Connect and find the target

Reuse the client's Flamigo connection or configure `https://api.flamigo.ai/mcp` with Streamable HTTP and browser OAuth (authorization code, S256 PKCE, token endpoint auth method `none`, MCP URL as resource indicator). Use `tools/list` for live schemas. For a connection check, call `list_presentations` and report whether access succeeded.

For an existing deck, use its presentation ID or find it by title with `list_presentations`. Read `flamigo://presentations/{presentation_id}` for its revision. Under that URI, read `/source` for original material, `/outline` for the saved outline, or `/slides` to map page numbers to slide IDs. Under `/slides/{slide_id}`, read the detail for its revision and preview state, `/dsl` for content, or `/html` for the saved preview. These reads suffice for requests to inspect or summarize existing work.

## Create the requested deliverable

For a new deck, call `create_presentation` with a title and the available source text. Keep its ID, revision, and `web_url`. Choose `presentation_style` (editorial, swiss, or minimal), `color_preset_id` and `font_preset_id` from the topic and audience. Read `flamigo://themes` for preset IDs. Set `illustration_instruction` to a fitting medium, palette, and level of detail.

| What the user has or wants | Action and deliverable |
| --- | --- |
| Notes; outline first | Call `draft_outline` on the saved source with `direction` and `max_pages` (1-30) as needed. Show the returned page sequence and stop for review. The page count is an upper bound. |
| Reviewed outline; generate content | Pass the outline as ordered `{page_no, sentence, layout}` items to `generate_slides_from_outline` with the current presentation revision. If the saved source is empty, Flamigo uses the outline sentences as source material. For a DSL-only request, return the generated content from each slide's `/dsl` resource and stop. |
| Complete DSL for every page | Call `generate_slides_from_dsl` with a nonempty `slides` array of `{dsl}` entries in display order. Both deck-generation tools replace **all existing pages and their checkpoints**. Use the single-page tools below for a partial edit. |
| Finished presentation | Starting from notes, draft the outline, generate the slide DSL, then render. For DSL already saved in the deck, go straight to rendering. Read `/slides`, call `render_slide` for each page with `has_html=false`, and read each returned `resource_uri` (also available on replay) to inspect previews for missing content, overflow, or unreadable diagrams. Share the deck's web URL. |
| Resume interrupted work | Call `get_generation_status` to inspect batch progress. Direct DSL replacement clears the old batch; imported pages start idle. Use `/slides` to find missing previews. For existing HTML, check `layout_content_current` on the slide detail before treating the preview as current. |
| Download a file | For the latest version, render missing previews and refresh existing previews whose slide detail reports `layout_content_current=false`. Read `/export/html` under the presentation URI and save the document. For PowerPoint, read `/export/pptx` (Pro) and save the decoded binary as a `.pptx` file. Return the saved file. |

## Change an existing presentation

| User intent | How to act |
| --- | --- |
| Add a slide | Call `create_slide` with the presentation revision. Omit `after_slide_id` to append or pass an anchor slide ID to insert. Use the new slide's ID and revision to supply content. |
| Shorten a title, fix data, or edit a component | Read the slide's `/dsl`, edit the requested elements, then send the complete revised DSL to `update_slide`. Call `render_slide` when the user wants the updated preview. |
| Rewrite a page from notes | Call `generate_slide_dsl` with `content` and the intended `layout`. It overwrites that slide's content and clears its layout and HTML. Render to produce a preview. |
| Refresh a preview or redesign a page | Use `render_slide` to refresh content and reuse a valid layout. Use `redraw_slide` to design a new layout from saved content, replace manual placements, and render it. For manual dragging or resizing, direct the user to layout editing in the web app. |
| Compare designs before choosing | Read the slide detail. If `layout_content_current` is false, render and read the new revision. Call `generate_slide_alternatives`, read the batch's `resource_uri`, and show ready candidates via their `preview_resource_uri`. Stop for a comparison-only request. To save a choice, call `apply_slide_alternative` with the batch and candidate IDs and the batch's `base_revision` as `expected_revision`. Regenerate expired choices; `fresh: true` reuses a batch still in progress. |
| Change the source, theme, or artwork | Use `update_presentation` for source text, presets, style, or illustration direction. New art follows `illustration_instruction`; existing illustrations are reusable DSL image components. For a single slide background, pass `background_image_url` to `update_slide`: HTTPS URL to set, empty string to clear, omission or null to keep. |
| Apply a new theme to existing previews | After `update_presentation`, use each `slides_to_refresh` entry's slide ID, revision and `apply_project_theme` flag with `render_slide`. If the user only requested a settings change, ask before refreshing existing previews. |
| Keep an undo point or restore a design | Call `save_checkpoint` before the change and retain the checkpoint ID. To restore, call `restore_checkpoint` with that ID and the current slide revision. Check the restored slide detail for preview freshness. |

## Write and retry

Use a new account-wide `operation_id` (1-128 characters after trimming) for each write. Pass the observed `expected_revision` of the target presentation or slide. After a transport failure, retry with the same ID and identical arguments. After a revision conflict, read the current state and use a new operation ID for the revised request.

## Example request

- Create six slides from these notes for a product review; show me the outline first.
