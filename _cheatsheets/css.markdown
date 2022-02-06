---
layout: post
title: CSS
---
Display flex.
- Направление и многострочность
  - Направление flex-direction: row(по умолчанию = inline-block)/column
  - Многострочность flex-wrap: nowrap(по умолчанию 1 строка)/wrap
  - Краткая запись направления и многострочности flex-flow: flex-direction, flex-wrap
    - initial (flex-flow: row, nowrap) значение по умолчанию
- Выравнивание:
  - По главной оси justify-content
  - По поперечной оси align-items
- Гибкость flex-элементов flex: flex-grow, flex-shrink, flex-basis
  - auto (flex: 1 1 auto;) при изменении контейнера размер пропорционально
  - none (flex: 0 0 auto;) при изменении контейнера размер не меняется
  - initial (flex: 0 1 auto;) значение по умолчанию, не увеличивается, уменьшается
