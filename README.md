# Simplified ARIA Annotations Proposal & Explainer


# Summary {#summary}

ARIA Annotations provide markup for accessible, in-page annotations, using experimental ARIA markup. This proposal is for markup targeted to become an integrated part of ARIA 1.3. It supersedes the current separate ARIA Annotations draft specification.

Annotations provide additional content related to a piece of content within a document. The most basic example is a description, which is possible today using aria-details. However, this explainer describes new markup that can be used to provide the annotation text, or to or provide the type of annotation.

The markup described has already been reviewed by a number of accessibility developers and stakeholders in key organizations, although no public signals have been provided. 


# Example Use Cases {#example-use-cases}


## Web Content use cases {#web-content-use-cases}



*   Document editor: suggestions, revisions, comments, footnotes and other authors present
*   Code editor: breakpoints, revision history, errors and warnings, etc.
*   Published content: attributions, footnotes and comments


## How Assistive Technology can utilize annotation semantics {#how-assistive-technology-can-utilize-annotation-semantics}



*   User issues command to navigate to previous/next annotated content (e.g. next commented text)
*   User issues command to navigate to previous/next annotation body (e.g. next comment)
*   User issues one of the above commands, but filtered by annotation purpose (e.g. navigate between text linked to a comment section, or navigate between comment sections)
*   User issues command to navigate from annotation body to the annotated content, or vice-versa
*   User adjusts setting for annotation verbosity in screen reader, e.g. all, shortened, none. Screen reader adjusts output, e.g. in shortened Braille "has commentary" becomes something like "\*ct", audio output utilizes an earcon/sound.


# Summary of proposed new markup {#summary-of-proposed-new-markup}



*   aria-description="[localized string]" (similar to how aria-label can be used instead of aria-labelledby. This is a generically useful attribute that has been requested for years, and can be placed on any element. [GitHub issue for aria-description](https://github.com/w3c/aria/issues/891).
*   role="suggestion"|"revision" -- used to group changes in a document (role "deletion" and "insertion" children).
*   role="mark" -- equivalent to HTML’s `<mark>`, to indicate highlighted text that has a special meaning or additional information tied to it (via aria-details=[id]). Future specific types of highlights could inherit from this, for example, code editor use cases could expand ARIA annotations to add breakpoint, error and warning roles. [GitHub issue for role="mark"](https://github.com/w3c/aria/issues/508). Annotated content may or may not be highlighted.
*   role="commentsection", inheriting from feed, to be used on a group of comments. When something has a related comment, it would use aria-details to point to the comment section. Individual comments would use role="comment", inheriting from "article", and supporting aria-level (in addition to the inherited aria-posinset and aria-setsize).


# Clarification of existing markup {#clarification-of-existing-markup}



*   aria-details can be used to tie any kind of annotation body to annotated content. A "description" is just the most basic kind of annotation body. Other types of annotation body purposes can be assigned by putting a role on the annotation body, such as doc-footnote, doc-endnote, definition or commentary.
*   A doc-footnote or doc-endnote should be linked to a doc-noteref using the existing aria-details relation property. 
*   A definition should be linked to a term using aria-details rather than aria-labelledby.


# Examples {#examples}


## Simple annotations -- no complex structure {#simple-annotations-no-complex-structure}

Simple annotations occur when there is no additional related section in the document that represents the annotation body, that the user might need to navigate to. The annotation is a localized text string (that may or may not correspond to a tooltip).


### Example -- Content is being edited by another author:

```html
<p aria-description="User bigbluehat is editing this nearby">Some text.</p>
```

In some cases, it may be helpful to provide a simple annotation purpose that is more specific than "definition". This can be done via aria-roledescription:



### Example -- A code editor provides a warning

```html
<mark aria-roledescription="warning" aria-description="Undefined global">myVar</mark>
```

(alternatively use `<span role="mark">` with the rest being the same).

In the future, commonly-used simple annotation purposes may be added to ARIA by creating new roles inheriting from "mark", e.g. "breakpoint". This would aid in filtering features of screen readers.


## Structured annotations {#structured-annotations}


Structured ARIA Annotations build upon the foundation of [aria-details](https://www.w3.org/TR/wai-aria-1.2/#aria-details), providing optional additions needed to enhance the reading experience of consuming annotated content via assistive technology.


The Accessible Rich Internet Applications [wai-aria-1.2] specification explains the use of aria-details as "[identifying] the element that provides a detailed, extended description for the object." ARIA Annotations build on that foundation and provide further contextual meaning via the use of role values specific to annotation use cases.


### 1. Example -- Definition of a word

Note: this goes against the current spec which suggests to use aria-labelledby. However, a description seems to be more correct.

```html
<p>Caesar’s army crossed the <term aria-details="rubicon-def">Rubicon</term>.</p>
<div role="definition">A shallow river in Italy. <a href="">See Wikipedia entry</a>.</p>
```

### 2. Example -- Footnote or endnote
```html
<p>Here is my footnote<span role="doc-noteref" aria-details="footnote1">1</span>.</p>
<p id="footnote1" role="doc-footnote">1. A footnote has little to do with actual feet.</p>
```

### Example -- Comment section for a specific piece of content
```html
<p>The <span aria-details="thread-1">cat is smart</span>.</p>
<div role="commentsection" id="thread-1">
  <div role="comment">
     As far as you can tell! Does she talk?
      <div role="comment">
         Yes, but only I understand her
      </div>
  </div>
  <form>
    <textarea placeholder="Reply here…"></textarea>
    <input type="submit">
  </form>
</div>
```

### Example -- Commentary section for entire article

```html
<article aria-details="all-comments">...</article>
<div id="all-comments" role="commentsection"...</div>
```

## Transient structured annotations {#transient-structured-annotations}

In some cases, structured annotations may only appear when a user navigates to an element. Unfortunately, this may make it difficult or impossible to implement an assistive technology command to navigate to other annotation bodies with a similar annotation purpose. 



### Example -- Rich tooltip in a code editor, linking other references to a symbol

```html
<div>String <span aria-details="catName-refs">catName</span>;</div>
<div id="catName-refs" role="tooltip">.. Several links to catName references ...</div>
```

## Content changes -- suggestions and revisions {#content-changes-suggestions-and-revisions}

A suggestion is essentially a non-accepted revision. It’s helpful to be able to differentiate them, e.g. so that a user can navigate only potential changes that haven’t been accepted yet.

A revision or suggestion used to group required children of insertion and deletion. They can have 1 insertion child, or 1 deletion child, or 1 of both (in any order). If content does not follow this structure correctly, ATs may not recognize and process the revision/suggestion correctly.

### Example -- Suggestion
```html
<p>The best pet is a <span role="suggestion"><span role="deletion">cat</span><span role="insertion">dog</span></span></p>
```

### Example -- Completed revision:
```html
<p>The best pet is a <span role="revision"><span role="deletion">cat</span><span role="insertion">dog</span></span></p>
```

Note: some browser implementations may need to special case a change from suggestion to revision, so that when a user accepts the suggestion, an accessible object is not unnecessarily destroyed and recreated, as may be done for other role changes.


## Combination: a content change with a structured annotation {#combination-a-content-change-with-a-structured-annotation}

Revisions and suggestions are commonly used used in conjunction with a structured annotation, whether auto generated or manually written by an author. In the case of suggestions, the commentary UI may include a UI control for accepting the suggestion.

This combines the markup from structured annotations and content changes.


### Example -- Suggestion with comment

```html
<p>The best pet is a <span role="suggestion" aria-details="comment-thread-1"><span role="deletion">cat</span><span role="insertion">dog</span></span></p>
<div id="comment-thread-1" role="commentary"><p>I think dogs are better</p><button>Accept</buton></div>
```

### Example -- Revision with additional attribution information

```html
<p>The best pet is a <span role="revision" aria-details="attribution-1"><span role="deletion">cat</span><span role="insertion">dog</span></span></p>
<div id="attribution-1">Modified by user aleventhal on 10/10/2020</div>
```

# Implementation Status and Instructions for Testing {#implementation-status-and-instructions-for-testing}


*   TBD: instructions for testing the additional markup
*   TBD: status of implementation and specification work


# FAQ {#faq}

Q. Why is aria-details used instead of aria-describedby? \
A. Unfortunately, many implementations of aria-describedby will simply flatten the pointed-to content into a description string, and does not allow a user to navigate to the pointed-to section. It is deemed important for the user to be able to navigate to structured annotations from the annotated content, or in the reverse direction.

Q. Why isn’t aria-description already part of the ARIA spec? \
A. Historically, it was thought that anything that would go into a description should also be accessible visually in the content. This has led to a widespread use of invisible aria-describedby sections, and many requests for the addition of an attribute like aria-description, analogous to aria-label and supporting a flat string.

Q. What if there are multiple structured annotations for a single piece of content? \
A. It’s possible  that aria-details should be expanded to allow multiple IDs. It currently only supports one. This is at least worth a discussion. \
 \
Q. How do ARIA Annotations relate to the Web Annotations Data Model? \
A. ARIA Annotations are deliberately more limited than what can be expressed within a Web Annotations Data Model [annotation-model] document. For example, ARIA Annotations cannot directly point to information external to the current Web page. However, they can point to a section in a web page that contains external links.

By design, ARIA Annotations do not contain as many semantic possibilities as the Web Annotation Data Model. However, any annotation can be mapped to the more generic ARIA annotations, if loss of some semantics is acceptable.


# Glossary {#glossary}

This explainer takes steps to use consistent terminology and modeling from the Web Annotation Data Model [[annotation-model](https://w3c.github.io/annotation-aria/#bib-annotation-model)].

**annotation**


An annotation is the combination of annotated content, one more of: an annotation purpose, a flat text text description, or a related annotation body.

**annotated content**


The annotated content is the range of text or other object which the annotation body is "about".


Note: In the [Web Annotations Data Model](https://www.w3.org/TR/annotation-model/#motivation-and-purpose), this is known as the **annotation target**. However, because the term "target" has a specific meaning in accessibility APIs as the end (and not beginning) of a relation, the term annotated content is preferred here, in order to avoid confusion with the actual directionality of `aria-details` in ARIA Annotation markup.

**annotation body**


The annotation body is visible information within a page which is connected to annotated content for a related purpose.

**annotation purpose**


The annotation purpose is the role the annotation body plays in relation to the annotated content (e.g. comment section vs. a footnote). \
TBD: explain the difference between this and when the role is put on the annotated content itself, e.g. role="mark".

