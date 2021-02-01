# Indicator Psuedo Element

*STATUS: EARLY DRAFT*

**Authors:**
* Brandon Ferrua, Salesforce
* Greg Whitworth, Salesforce

## Overview
Native controls and components have been a long-standing issue for [web developers and designers](http://gwhitworth.com/blog/2019/07/form-controls-components/). A few of the more commonly re-created controls are `<input type="checkbox">` and 
`<input type="radio">`. 

*Note: All examples in this document will utilize the checkbox for consistency, but the radio anatomy of the indicator itself is the same and thus is included in the normative spec text*

To enable the capability that authors seek it requires a definition of new parts and a way to ensure an interopable approach to style them.

## the `base` keyword value for the `appearance` property
User-agents are encouraged to provide their own solution that matches their own preferred design systems. There are often valid arguments for a user-agent's built controls/components to match the OS they are rendered on.
However, if the author wishes to make a built-in control or component match their site's design the author must re-create it. This not only is a poor developer experience but can also lead a degraded user-experience.

In order to allow an author to opt-in to being able to customize a built-in component or control we're proposing a `base` keyword value to the [appearance](https://drafts.csswg.org/css-ui-4/#appearance-switching) property. This will inform the 
user-agent to load in the standardized styles for the control/component in order to ensure an interoperable starting point 
for authors to begin styling from.

<dl>
    <dt>
        <dfn><code>base</code></dfn>
        <dd>Similar to the <a href="https://drafts.csswg.org/css-ui-4/#ref-for-valdef-appearance-none" target="_blank">none</a> value; the element is rendered following the usual rules of CSS. Replaced elements other than <a href="https://drafts.csswg.org/css-ui-4/#widget" target="_blank">widgets</a> are not affected by this and remain replaced elements. The control, will then load in the standardized 
        graphic representation and base stylesheet shipped by all user agents.</dd>
    </dt>
</dl>


## the `::indicator` pseudo-element
In order to enable authors to adjust the user agent provided checkbox and radio input types, we're proposing 
the `indicator` pseudo-element. This pseudo-element provides a visual representation of a checkbox's or radio`s current state.

When paired with the `:checked` and `:indeterminate` pseudo-classes an author can adequately provide the 
end-user with a custom tailored experience for all possible states of each control.

## Use cases
The [accent-color proposal](https://github.com/mfreed7/accent-color/blob/master/proposal.md#input-typecheckbox) highlighted a need from authors that wanted to utilize the natively shipping controls but adjust the colors to match their site's design. Currently, in order to achieve this, the author will need to set `appearance: none` on the input and provide a replacement checkbox that they have created, often using additional assets from graphic software. 

# Putting it all together

In order to achieve the necessary end result for the author, it requires the combination of the `base` keyword, 
which instantiates the `base` origin. Upon that instantiation it has a profound impact on the `indicator` pseudo-element. 

If the `appearance` property computes to `base` for the following elements:
* [HTMLInputElement](https://html.spec.whatwg.org/multipage/input.html#the-input-element) in a [Checkbox state](https://html.spec.whatwg.org/multipage/input.html#checkbox-state-(type=checkbox))
* [HTMLInputElement](https://html.spec.whatwg.org/multipage/input.html#the-input-element) in a [Radio Button state](https://html.spec.whatwg.org/multipage/input.html#checkbox-state-(type=radio))

The user-agent **MUST** make the immediate child of the `::indicator` pseudo-element the following SVG:

```
<svg data-name="Layer 1" viewBox="0 0 187.44 231.39" width="100%" height="100%" xmlns="http://www.w3.org/2000/svg">
    <path d="m7.54 136.22 58.3 76.55s39-110.33 114.15-207"/>
</svg>
```

And the following styles **MUST** be placed in the user-agent's stylesheet:

```
input[type=checkbox] {
    display: flex;
    width: 13px; 
    height: 13px;
    border: 1px solid #888;
    box-sizing: border-box;
    fill: none;
    stroke: #231f20;
    stroke-miterlimit: 10;
    align-items: center;
    justify-content: center;
}

input[type=checkbox]::indicator {
    display: none;
}

input[type=checkbox]::indicator,
input[type=checkbox]:checked::indicator {
    width: 6px;
    height: 6px;
    stroke-width: 60px;
    align-items: center;
    justify-content: center;
    margin: -2px 0 0 -2px;
}

/* MORE STYLES TO BE DEFINED AT A LATER DATE */

```

By providing an interoperable base stylesheet, anatomy and default graphic this enables the pseudo-element fully styleable. 
Below are some examples of this in practice:

![Examples of the indicator pseudo-element in practice](indicator-examples.png)

**Note:** *These examples can be seen in this Codepen Project [here](https://codepen.io/gregwhitworth/project/full/DkGOoY). It was simply built for styling explorations 
and not to be a production ready checkbox. And while only showing checkbox, the anatomy of the radio indicator itself is the same as a checkbox (not the radio group, but the indicator itself)*

## Open Questions

* Should the base.css stylesheet be a new standardized specification in the CSS WG that is available for download as a .css file for download by user-agents?
* How should focus rects be handled (current proposal is to leave them as user-agent defined until focus-rect primitives are added to the platform to achieve similar experiences currently shipping)?
* The `d` attribute is a presentational and so technically it can be used as a way to change the graphic that is rendered. However it doesn't produce a box and thus many usecases are lost. How do we want to handle this?

## Other Solutions Considered

### Unicode
One consideration, which is often used by authors in the `::before` or `::after` pseudo-element is to utilize the 2713 unicode code point for checkboxes. Using this unicode code point is an elegant proposal in that all browsers on all operating systems currently use the same graphic provided by the OS; thus achieving interoperability. Below is a reference comparison across iOS, Android, Windows and MacOS and various browsers.

![A reference of different checkmarks across different browsers](unicode-checkmark-comparison.png)

While this is a solid start and would make implementation much simpler this unfortunately does not enable many of the 
designs that are found across the web today as you can only propogate certain CSS properties, such as `color `in order to modify the color but you can't adjust strokes, joints, fills to the extent that can be done with SVG. Due to these limitations this direcion was abandoned.

### Background images
When initially exploring the password-reveal pseudo-element we defined it to be a background image. This would allow the author to replace 
the graphic using an image or the Houdini `paint()` method. This has negative implications in that it came with the following constraints:

* **Unknown parent styles & anatomy:** While the pseudo element and application of the graphic would be standard. The author would have no insight into how its ancestor tree would be structured nor styled. One concrete example of this was trying to replicate a scenario where the pseudo-element would reside outside of the input but there was an ancestor that couldn't be reached between the pseudo-element & the input. This resulted in the psuedo-element being clipped.
* **Needing graphic design capabilities:** Using a background-image unlocked some scenarios but it still required an author to produce and asset to replace it. Looking at the accent-color scenarios again the authors had no issues with the graphic but simply the colors. Forcing them to create a graphic to meet their needs just perpetuates the problem.

## Acknowledgements

* Melanie Richards, Microsoft
* Tab Atkins, Google