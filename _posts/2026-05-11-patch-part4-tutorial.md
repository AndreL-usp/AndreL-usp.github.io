---
title: "Contribuição 1: Parte 4 - Feedback"
date: 2026-05-11
---

## Feedback dos Mantenedores

Após reenviar, finalmente nosso patch foi revisado e, aparentemente, por um dos principais contribuidores dos drivers da AMD no kernel do linux, Alex Deucher!!!

O feedback foi o seguinte:

> "This is not common to jpeg v5.  jpeg_v5_0_0.c has a different interrupt handler, although it could probably be shared with that as well.  I think the additional fields in the IH should be 0s so it should work correctly, but we'd need to verify.  If you do want to share this, I would just make the implementation in jpeg_v5_0_1.c non-static and then just use it directly in jpeg_v5_0_2.c."