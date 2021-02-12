# Tag, Attributes, then Classes methodology
**Tag, Attributes, then Classes** or **TAC** helps you solve the four main problem areas of implementing a design: 
- generic repeating styles
- UI components
- evolution and maintenance
- sharing the code

TAC is different than other class-based CSS methodologies (BEM, OOCSS, utility-only Tailwind).

TAC can be applied to simple sites, large apps, and full-scale design systems.

TAC leverages the web platform in ways that take advantage of everything it has to offer.

TAC results in small, standards-based code with zero dependencies.

TAC works with everything - any framework or no framework.

If you want to see a design system built following the TAC methodology, check out [M-](https://m-docs.org). You can also read [10 Ways M- Raises the Bar for UI Libraries](https://dev.to/jfbrennan/10-ways-m-raises-the-bar-for-ui-libraries-2p4i) and follow [@realEmDash](https://twitter.com/realEmDash).

### Outline
1. Create utility classes that do one generic thing
  - Avoid single character abbreviations
  - Prefix related classes, e.g. `.txt-center`, `.txt-right`
  - The `.lowercase-dash-separated` convention is all you need
1. Define [custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) for design tokens 
1. Use tags and attributes for UI components
  - Use up HTML, then imitate it
  - Define a custom tag prefix
  - Give tags short but meaningful names
  - Use attributes for component variations
  - Specificity scores increase in tag -> utility class -> tag+attribute order
1. Evolve your components with Custom Elements
  - Components always start life as a CSS-only custom tag
  - Upgrade to Custom Elements when JavaScript is required
  - Upgrading and downgrading is non-breaking
1. TAC is ideal for shared design systems
  - Framework-agnostic
  - Static sites, SPA, SSR, PWA - they're all supported
  - Value is in the adoption rate


