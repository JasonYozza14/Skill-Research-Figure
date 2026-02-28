---
name: research-figure
description: >
  Generate publication-quality LaTeX TikZ method pipeline/flowchart figures for academic papers.
  Use this skill whenever the user wants to create a method overview figure, pipeline diagram,
  architecture diagram, or system flowchart for their research paper. Also trigger when the user
  describes their paper's method and wants a visual representation, mentions "method figure",
  "pipeline figure", "architecture diagram", "flow diagram", "方法流程图", "画图", or wants to
  visualize their approach/framework/system in a paper. Even if the user just says "draw my method",
  "I need a figure for my paper", "help me make a figure", or describes a pipeline and asks for a
  diagram, this skill should activate.
---

# Research Figure: LaTeX TikZ Method Pipeline Generator

Generate publication-quality method pipeline figures through a streamlined conversation: understand the method, confirm structure, generate TikZ code, compile, and iterate.

## Design Principles

These principles come from best practices in academic figure design:

1. **Card-based layout**: Every module is enclosed in a rounded rectangle (card). Cards are connected by arrows. This makes the pipeline structure immediately clear.
2. **Low-saturation colors**: Use muted, pastel colors. Never use vivid/saturated colors. The figure should look professional, not like a PowerPoint.
3. **Compact layout**: Minimize whitespace. Align elements horizontally and vertically. The figure should be tight and dense, not scattered.
4. **Sans-serif fonts**: Use `\sffamily` throughout for a modern, clean look.
5. **Highlight contributions**: The user's novel contribution modules should be visually distinct (thicker border, different background color) from standard modules.
6. **2D visualization elements**: Replace plain text with stylized visual elements where possible (feature maps as stacked rectangles, network blocks as labeled boxes, etc.). Keep these elements simple and 2D — no 3D rendering needed.

## Workflow

### Phase 1: Understand the Method

When the user provides a description (abstract, method summary, or even a vague idea):

1. Read the description carefully and extract the pipeline structure as a chain:
   `Input → Module1 → Intermediate1 → Module2 → ... → Output`
2. For each module, identify:
   - **Name**: a short descriptive name
   - **Type**: data processing, neural network, optimization, inference, loss, etc.
   - **Is novel?**: whether this is part of the user's contribution
   - **Internal elements**: what happens inside (convolution, attention, MLP, etc.)
3. Identify the overall flow pattern:
   - Linear (most common)
   - Loop/iterative
   - Multi-branch (parallel processing)
   - Two-stage (train + inference, or two sub-problems)

### Phase 2: Confirm with User (1-2 rounds)

Present the extracted structure to the user in a clear, structured format using AskUserQuestion or direct conversation. Keep this brief — the goal is to confirm, not to design from scratch.

**Round 1 (required):** Show the pipeline structure and ask:
- Is the module breakdown correct?
- Which modules are the key contributions?
- Layout preference: horizontal (default), vertical, loop, two-stage, or multi-branch?
- Color scheme preference? (If not specified, default to Blue-Gray)

Format the pipeline clearly, for example:
```
Extracted pipeline:
  Image Input → Feature Encoder → [YOUR METHOD: Adaptive Fusion Module] → Decoder → Output Mesh

Layout: Linear horizontal (left to right)
Highlight: "Adaptive Fusion Module" (novel contribution)
Color scheme: Blue-Gray (default)
```

**Round 2 (only if needed):** If the pipeline is complex or the user has special requests for internal module details, ask about:
- Specific visualization elements inside modules
- Special annotations or labels
- Loss functions or training signals to show

If the method is straightforward, skip Round 2 and proceed directly to generation.

### Phase 3: Generate TikZ Code

Read the reference files to assemble the figure:

1. **Read `references/layout_templates.md`** — pick the template matching the layout type (linear horizontal, vertical, loop, two-module, multi-branch). Use the template as the structural skeleton.

2. **Read `references/color_schemes.md`** — pick the user's chosen color scheme (or Blue-Gray default). Copy the `\definecolor` block into the preamble.

3. **Read `references/tikz_elements.md`** — select element styles needed:
   - Module box styles (standard, novel, optional, io, group, loss)
   - Arrow styles (standard, labeled, orthogonal, dashed, bidirectional)
   - 2D elements (feature maps, network blocks, grids, image placeholders)
   - Annotations (braces, callouts, dimension labels)

4. **Compose the final `.tex` file** following these rules:
   - Use `\documentclass[border=5pt]{standalone}` for a cropped output
   - Include all necessary packages and TikZ libraries
   - Define all colors in the preamble
   - Define all styles with `\tikzset` in the preamble
   - Place nodes using `positioning` library (e.g., `right=1.0cm of nodeA`)
   - Keep spacing consistent: typically 0.8–1.2cm between nodes
   - Ensure all text fits within its box (use `\\` for line breaks if needed)
   - Use `\textbf` for module titles, `\footnotesize\color{textLight}` for subtitles
   - Add arrow labels for important data flows (features, latent codes, etc.)
   - Use math mode for symbols: `$\mathbf{z}$`, `$\mathcal{L}$`, etc.

5. **Write the `.tex` file** to the working directory (e.g., `figure.tex` or a user-specified name).

### Phase 4: Compile and Self-Check

1. **Compile** using the bundled script:
   ```bash
   bash .claude/skills/research-figure/scripts/compile_tikz.sh <path-to-tex-file>
   ```

2. **If compilation fails**: read the error output, fix the LaTeX code, and retry. Common issues:
   - Missing packages → add `\usepackage{...}`
   - Undefined color → check `\definecolor` definitions
   - Node positioning errors → check node names and `of` references
   - Math mode errors → check `$...$` matching

3. **View the result**: Use the Read tool to view the generated PNG image.

4. **Self-check the figure** against this checklist:
   - [ ] All text is readable (no overlapping)
   - [ ] Arrows point in the correct direction
   - [ ] Horizontal and vertical alignment is clean
   - [ ] Layout is compact (no large empty areas)
   - [ ] Color scheme is consistent and harmonious
   - [ ] Novel contribution modules are visually highlighted
   - [ ] Module names match what was agreed with the user
   - [ ] Font is sans-serif throughout

5. **If issues found**: fix the TikZ code and recompile. Do NOT show a broken figure to the user.

### Phase 5: Present and Iterate

1. Show the compiled figure to the user (the PNG image).
2. Explain the structure briefly.
3. Ask if any changes are needed.
4. Based on feedback, modify the TikZ code, recompile, and show again.
5. Repeat until the user is satisfied.
6. Provide the final `.tex` file path so the user can:
   - Include it directly in their paper with `\input{}`
   - Or compile it standalone and include via `\includegraphics{}`

## Important Notes

- Always generate **complete, standalone** `.tex` files that compile on their own. Never output partial snippets.
- The figure should be designed for **academic papers** — clean, professional, understated. Not flashy.
- When in doubt about layout details, choose the simpler option. Less is more.
- If the user's method is very complex, consider breaking it into sub-figures or simplifying the visualization. A cluttered figure is worse than a simplified one.
- The `standalone` document class with `border=5pt` produces a tightly cropped PDF, perfect for `\includegraphics`.
- For figures wider than typical column width (~8.5cm), suggest using a `figure*` environment (spanning two columns).

## Common TikZ Pitfalls

When generating TikZ code, avoid these common mistakes:

1. **Duplicate node names**: In TikZ, `\node[...] (name) ...` renders a visible node each time it is called, even if `name` is reused. Only the *last* definition is reachable by name — earlier ones become orphaned but still visible on the canvas. **Fix**: Use `\coordinate` for temporary positioning points, and only create the final `\node` once.

2. **`fit` node center offset**: A `fit` node's center is determined by the bounding box of *all* fitted elements. If children are asymmetric (e.g., a title sits above the content), the center shifts. Arrows drawn to/from the fit node's center will therefore not be horizontal/vertical as expected. **Fix**: Either add symmetric anchor points (mirror the title offset below), or use explicit anchor-based arrow endpoints like `(encoder.east) -- (nerf.west |- encoder.east)` to force alignment.

3. **Diagonal arrows between non-aligned nodes**: When two nodes are not on the same row or column, a plain `--` connection produces a diagonal line, which looks unprofessional. **Fix**: Use orthogonal routing with `rounded corners` and the `|-` or `-|` path operators. Example:
   ```latex
   \draw[stdarrow, rounded corners=4pt] (nodeA.south) -- ++(0, -0.5cm) -| (nodeB.north);
   ```

4. **Multi-branch merge node positioning**: When placing a merge node (e.g., `+` or `⊕`) for parallel branches, the node's x-coordinate should align with (or extend beyond) the rightmost branch endpoint, and the y-coordinate should be the midpoint between the branches. Use `\coordinate` for intermediate calculations, then position the merge node with `|-` syntax. Route arrows from each branch using orthogonal paths (`|- merge.155` / `|- merge.205`) for clean connections.

5. **Orthogonal offset must clear group box padding**: When drawing an orthogonal path between two group boxes (e.g., `(outA.south) -- ++(0, -Xcm) -| (inB.north)`), the vertical offset `X` must account for the group box's `inner sep` (typically 10pt ≈ 0.35cm). If the offset is too small, the horizontal segment will be visually occluded by the group box boundary. **Fix**: Set the offset to at least half the gap between the two group boxes. For example, with `below=2.0cm` spacing and `inner sep=10pt`, the visible gap is roughly `2.0 - 2×0.35 ≈ 1.3cm`, so use `-1.0cm` (not `-0.5cm`) to place the horizontal segment in the middle of the gap.

6. **Fork/merge routing asymmetry**: When fixing orthogonal routing on one side of a fork-merge pattern (e.g., the merge side: `procA → merge`), always check the other side (the fork side: `input → brA/brB`) for the same diagonal-line problem. Both sides must use consistent orthogonal routing with `rounded corners` and `|-` or `-|` operators. A common mistake is fixing the merge arrows but forgetting the fork arrows remain as plain `--` diagonals.
