# Clothes Research

Research project on how **clothing / costumes** are handled in a **manga-to-anime pipeline**,
covering both the traditional studio workflow and AI-assisted approaches.

## Scope

1. **Industry workflow** – costume settei (model sheets), simplification rules, color design, sakkan consistency.
2. **Extraction from manga** – character detection, clothing segmentation, tagging, screentone handling, colorization.
3. **Generative AI consistency** – LoRA / IP-Adapter / ControlNet, outfit transfer, video models, flicker control.
4. **Cloth animation** – 2D hand-drawn, Live2D-style rigs, AI in-betweening, 3D cel-shaded cloth simulation.
5. **Building the pipeline** – reference architecture, outfit database schema, compute costs, legal/licensing.

## Layout

```
research_notes/<title>/   raw notes from each research track
reports/<title>.md        final synthesized report
```

## Reports

- [Clothes in manga to anime pipeline](reports/Clothes%20in%20manga%20to%20anime%20pipeline.md)
- [Solo 3D anime clothes in Unreal](reports/Solo%203D%20anime%20clothes%20in%20Unreal.md) – one-person, no-training plan for clothing an existing 3D body in UE5
- [Automatic MD like garment generator](reports/Automatic%20MD%20like%20garment%20generator.md) – design for an automatic, MD-like image/profile → sewing pattern → draped 3D garment system

## Guides

- [Marvelous Designer workflow](guides/Marvelous%20Designer%20workflow.md) – step-by-step MD → Blender → Unreal for an existing anime body
