# Indicator Psuedo Element

*STATUS: EARLY DRAFT*

**Authors:**
* Greg Whitworth, Salesforce

## Overview
Native controls and components have been a long-standing issue for [web developers and designers](http://gwhitworth.com/blog/2019/07/form-controls-components/). A few of the more commonly re-created controls is `<input type="checkbox">` and 
`<input type="radio">`.

Below are a few examples from across design systems and component libraries:

# INSERT IMAGES HERE

In order to enable the capability that authors are seeking it requires a definition of new parts and a way in which to ensure and interopable approach to style on top of so that an author doesn't rely on the default styles of their preferred user-agent.

## the `base` keyword value for the `appearance` property
User-agents are encouraged to provide their own solution that matches their own preferred design systems. There are 
often valid arguments for a user-agent's builth-in controls/components to match that of the OS they are rendered on.
However, if the author wishes to make a built-in control or component match their site's design the author has to re-create it. This not only is a poor developer experience but can also lead a degraded user-experience.

In order to allow an author to opt-in to being able to customize a built-in component or control we're proposing a `base` keyword value to the [appearance](https://drafts.csswg.org/css-ui-4/#appearance-switching) property. This will inform the 
user-agent to load in the standardized styles for the control/component in order to ensure an interopable starting point 
for authors to begin styling from.

<dl>
    <dt>
        <dfn><code>base</code></dfn>
        <dd>Similar to the <a href="https://drafts.csswg.org/css-ui-4/#ref-for-valdef-appearance-none" target="_blank">none</a> value; the element is rendered following the usual rules of CSS. Replaced elements other than <a href="https://drafts.csswg.org/css-ui-4/#widget" target="_blank">widgets</a> are not affected by this and remain replaced elements. The control, will then load in the standardized 
        graphic representation and base stylesheet shipped by all user agents.</dd>
    </dt>
</dl>

## The base stylesheet
The CSS cascade specification defines various [origins](https://www.w3.org/TR/css-cascade-3/#cascading-origins) that determine precedance of cascade order. In order to achieve interoperability across operating systems and form 
factors we are proposing a `base` origin.

<dl>
    <dt>
        <dfn><code>Base Origin</code></dfn>
        <dd>A standardized base stylesheet shipped by all user-agents.</dd>
    </dt>
</dl>

The `Base Origin` will be added to the [cascade sorting order](https://www.w3.org/TR/css-cascade-3/#ref-for-origin%E2%91%A1) to reflect the way in which the cascade will change.

1. Transition declarations
2. Important base declarations
3. Important user agent declarations
4. Important user declarations
5. Important author declarations
6. Animation declarations
7. Normal author declarations
8. Normal user declarations
9. Normal user agent declarations
10. Normal base declarations


## the `::indicator` psuedo element
In order to enable authors to adjust the user agent provided checkbox and radio input types, we're proposing 
the `indicator` psuedo element. This psuedo element provides a visual representation of a checkbox's or radio`s current state.

When paired with the `:checked` and `:indeterminate` pseudo classes an author can adequately provide the 
end user with a custom tailored experience for all possible states of each control.

## Use cases
The [accent-color proposal](https://github.com/mfreed7/accent-color/blob/master/proposal.md#input-typecheckbox) highlighted a need from authors that wanted to utilize the natively shipping controls but adjust the colors to match their site's design. Currently, in order to achieve this, the author will need to set `appearance: none` on the input and provide a replacement checkbox that they have created, often using additional assets from graphic software. 

### Example:

# INSERT EXAMPLE HERE

*NOTE: I have a rough web component that utilizes the pseudo indicator and replicates the base origin [here](https://codepen.io/gregwhitworth/project/full/DkGOoY). I'll be adding in examples that recreate the prior ones and list any potential limitations that are identified with this approach at this stage.*

## Open Questions

* OPEN QUESTIONS HERE

## Other Solutions Considered

### Unicode
One consideration, which is often used by authors in the `::before` or `::after` pseudo elements is to utilize the 2713 unicode code point for checkboxes. This is an elegant proposal in that all browsers on all OSes currently use the same graphic from the OS; thus achieving interoperability on that OS. Below is a reference comparison across iOS, Android, Windows and MacOS and various browsers.

![A reference of different checkmarks across different browsers](unicode-checkmark-comparison.png)

While this is a solid start and would make implementation much simpler this unfortunately does not enable many of the 
designs that are found across the web today.

## Acknowledgements

* Brandon Ferrua, Salesforce
* Melanie Richards, Microsoft