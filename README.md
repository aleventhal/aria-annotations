# Simplified ARIA Annotations Proposal & Explainer


# Summary

ARIA Annotations provide markup for accessible, in-page annotations, using experimental ARIA markup. This proposal is for markup targeted to become an integrated part of ARIA 1.3. It supersedes the current separate ARIA Annotations draft specification.

Annotations provide additional content related to a piece of content within a document. The most basic example is a description, which is already specified today using aria-details. However, this explainer describes new markup that can be used to provide the annotation text (aria-description), or to provide more semantic information about the annotation. There is a parallel effort to work with assistive technology vendors to work on their implementations for both the existing and new markup.

Please add feedback on this proposal to the [ARIA Annotations GitHub issue #1109](https://github.com/w3c/aria/issues/1109).

# Example Use Cases


## Web Content use cases



*   Document editor: suggestions, revisions, comments, footnotes and other authors present
*   Code editor: breakpoints, revision history, errors and warnings, code comments, etc.
*   Published content: attributions, footnotes and comments


## How Assistive Technology can utilize annotation semantics



*   User issues command to navigate to previous/next annotated content (e.g. next commented text)
*   User issues command to navigate to previous/next annotation body (e.g. next comment)
*   User issues one of the above commands, but filtered by annotation purpose (e.g. navigate between text linked to a comment or navigate between comments)
*   User issues command to navigate from annotation body to the annotated content, or vice-versa
*   User adjusts setting for annotation verbosity in screen reader, e.g. all, shortened, none. Screen reader adjusts output, e.g. in shortened Braille "has comments" becomes something like "\*ct", audio output utilizes an earcon/sound.


# Summary of proposed new markup



*   aria-description="[localized string]" (similar to how aria-label can be used instead of aria-labelledby). This is a generically useful attribute that has been requested for years, and can be placed on any element. \
[GitHub issue for aria-description](https://github.com/w3c/aria/issues/891). \
[Pull request for aria-description](https://github.com/w3c/aria/pull/1137). \
Note for authors: aria-description should not be used when there is a more specific semantic to express the same information. \
Order of precedence for description fields in accessibility APIs: 1) aria-describedby, 2) aria-description, 3) native markup such as HTML's @title
*   role="suggestion" -- used to group proposed changes in a document (role "deletion" and "insertion" children). \
[Pull request for role="suggestion"](https://github.com/w3c/aria/pull/1134).
*   role="mark" -- equivalent to [HTML’s `<mark>`](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-mark-element), to indicate highlighted text that has a special meaning or additional information tied to it (via aria-details=[id]). Future specific types of highlights could inherit from this, for example, code editor use cases could expand ARIA annotations to add breakpoint, error and warning roles. Note: annotated content is not always highlighted. \
[GitHub issue for role="mark"](https://github.com/w3c/aria/issues/508). \
[Pull request for role="mark"](https://github.com/w3c/aria/pull/1133).
*   role="comment" -- Content would point to a related comment via aria-details. Replies to comments would be child elements or virtual children via aria-owns. The level of the comment, setsize and posinset should be automatically exposed by the browser. The author can use aria-level, aria-setsize an aria-posinset to override these computations. \
[Pull request for role="comment"](https://github.com/w3c/aria/pull/1135).


# Changes/clarification to existing markup



*   aria-details can be used to tie any kind of annotation body to annotated content. A "description" is just the most basic kind of annotation body. Other types of annotation body purposes can be assigned by putting a role on the annotation body, such as doc-footnote, doc-endnote, definition or comment.
*   aria-details should support IDREFS as other relations do, otherwise it's unclear what an author should do when there happens to be multiple, unrelated annotations for the same piece of content. \
[Pull request for changes to aria-details](https://github.com/w3c/aria/pull/1136). \
This will require asking ATs to support this in their navigation scheme.
*   A doc-footnote or doc-endnote should be linked to a doc-noteref using the existing aria-details relation property. 
*   A term should be linked to its defintion using aria-details rather than aria-labelledby.


## Simple annotations -- no complex structure

Simple annotations occur when there is no additional related section in the document that represents the annotation body, that the user might need to navigate to. The annotation is a localized text string (that may or may not correspond to a tooltip).


### Example -- Content being edited by another author

This can occur in a multi-authoring environment, such as Google Docs, which may wish to mark a piece of text as being currently edited by another author.

```html
<p aria-description="User bigbluehat is editing this nearby">Some text.</p>
```



### Example -- A code editor provides a warning

In some cases, it may be helpful to provide more information about the purpose of a highlight. This can be done via aria-roledescription.

For example, an online code editor may use highlighting and tooltips to indicate code warnings.

```html
<mark aria-roledescription="warning" aria-description="Undefined global">myVar</mark>
```

Alternatively, rather than html's `<mark>`, the content can use `<span role="mark">` with the rest being the same.

In the future, commonly-used highlight purposes may be added to ARIA by creating new roles inheriting from "mark", e.g. "breakpoint". This would aid in filtering features of screen readers.


## Structured annotations


Structured ARIA Annotations build upon the foundation of [aria-details](https://www.w3.org/TR/wai-aria-1.2/#aria-details), providing optional additions needed to enhance the reading experience of consuming annotated content via assistive technology.


The Accessible Rich Internet Applications [wai-aria-1.2] specification explains the use of aria-details as "[identifying] the element that provides a detailed, extended description for the object." ARIA Annotations build on that foundation and provide further contextual meaning via the use of role values specific to annotation use cases.


### Example -- Definition of a word

Note: this goes against the current spec which currently suggests to use aria-labelledby. However, a description is likely a better match, as an aria-labelledby affects the accessible name of an object.

```html
<p>Caesar’s army crossed the <term aria-details="rubicon-def">Rubicon</term>.</p>
<div role="definition" id="rubicon-def">A shallow river in Italy. <a href="#">See Wikipedia entry</a>.</p>
```

### Example -- Footnote or endnote

Currently, footnotes and endnotes are defined by DPUB, but there is no good way to link a note reference number to the actual note.

```html
<p>
  Here is my footnote<span role="doc-noteref" aria-details="footnote1">1</span>.
</p>
<p id="footnote1" role="doc-footnote">
  1. A footnote has little to do with actual feet.
</p>
```

### Example -- Comments on a specific piece of content

Comments are special and worth providing special semantics for, to enable navigation between comments.
Some comments may contain a tree structure of replies.

In this example, a comment is linked to a small piece of text within the document.

```html
<p>The <span role="mark" aria-details="thread-1">cat is smart</span>.</p>
<div role="comment">
   As far as you can tell! Does she talk?
    <div role="comment">
       Yes, but only I understand her
    </div>
  <form>
    <textarea placeholder="Reply here…"></textarea>
    <input type="submit">
  </form>
</div>
```

### Example -- Comments for entire article

In this example, an article has a comment section beneath it. The entire comment section can be linked to the article. The reply structure can be implemented via DOM descendants or aria-owns.

```html
<article aria-details="all-comments">...</article>
<div role="section" id="all-comments">
  <div role="comment">
    Comment
    <div role="comment">
      Reply 1
    </div>
    <div role="comment">
      Reply 2
    </div>
  </div>
</div>

```

## Transient structured annotations

In some cases, structured annotations may only appear when a user navigates to an element. Unfortunately, this may make it difficult or impossible to implement an assistive technology command to navigate to other annotation bodies with a similar annotation purpose. 


### Example -- Rich tooltip in a code editor, linking other references to a symbol

```html
<div>String <span aria-details="catName-refs">catName</span>;</div>
<div id="catName-refs" role="tooltip">.. Several links to catName references ...</div>
```

## Content changes -- suggestions and revisions

A revision is a change that has already occured within a document, whereas a suggestion is a potential change in a document. In both cases, the change or potential change is represented by 1 insertion element, or 1 deletion element, or 1 of each (adjacent, and in either order). These can be indicated using `<ins>`/`<del>` or `role="insertion"|"deletion"`. If content does not follow this structure correctly, ATs may not recognize and process the suggestion/revision correctly. 

The suggestion role is used on the parent of the change element(s) to differentiate the change from a revision, so that a user can navigate only potential changes that haven’t been accepted yet. However, a revision itself does not require any particular role to parent the change, as the existence of the adjacent insertion and deletion elements already represent a revision.

When a suggestion is accepted the role on the containing element should be removed, effectively changing it into a revision.

### Example -- Revision

```html

<p>
  The best pet is a
  <span role="deletion">cat</span>
  <span role="insertion">dog</span>
</p>
```

### Example -- Suggestion


```html
<p>
  The best pet is a
  <span role="suggestion">
    <span role="deletion">cat</span>
    <span role="insertion">dog</span>
  </span>
</p>
``


Technical note: some browser implementations may need to special case a change from suggestion to revision, so that when a user accepts the suggestion, an accessible object is not unnecessarily destroyed and recreated, as may be done for other role changes.


## Combination: a content change with a structured annotation

Revisions and suggestions are commonly used in conjunction with a structured annotation, whether auto generated or manually written by an author. In the case of suggestions, the annotation may be a comment, and may include a button for accepting the suggestion.


### Example -- Suggestion with comment

```html
<p>
  The best pet is a
  <span role="suggestion" aria-details="comment-thread-1">
    <span role="deletion">cat</span>
    <span role="insertion">dog</span>
  </span>
</p>
<div id="comment-thread-1" role="comment">
  <p>Suggested change from Frederico"</p>
  <p>I think dogs are better</p>
  <button>Accept</buton>
  <div role="comment">
    <p>My reply.</p>
  </div>
</div>
```

### Example -- Revision with additional attribution information

```html
<p>
  The best pet is a
  <span aria-details="attribution-1">
    <span role="deletion">cat</span>
    <span role="insertion">dog</span>
  </span>
</p>
<div id="attribution-1">
  Modified by user aleventhal on 10/10/2020
</div>
```

# FAQ

Q. Why is aria-details used instead of aria-describedby? \
A. Unfortunately, many implementations of aria-describedby will simply flatten the pointed-to content into a description string, and does not allow a user to navigate to the pointed-to section. It is deemed important for the user to be able to navigate to structured annotations from the annotated content, or in the reverse direction.

Q. Why isn’t aria-description already part of the ARIA spec? \
A. Historically, it was thought that anything that would go into a description should also be accessible visually in the content. This has led to a widespread use of invisible aria-describedby sections, and many requests for the addition of an attribute like aria-description, analogous to aria-label and supporting a flat string.

Q. How do ARIA Annotations relate to the Web Annotations Data Model? \
A. ARIA Annotations are deliberately more limited than what can be expressed within a Web Annotations Data Model [annotation-model] document. For example, ARIA Annotations cannot directly point to information external to the current Web page. However, they can point to a section in a web page that contains external links.

By design, ARIA Annotations do not contain as many semantic possibilities as the Web Annotation Data Model. However, any annotation can be mapped to the more generic ARIA annotations, if loss of some semantics is acceptable.


# Glossary

This explainer takes steps to use consistent terminology and modeling from the Web Annotation Data Model [[annotation-model](https://w3c.github.io/annotation-aria/#bib-annotation-model)].

<dl>
<dt>annotation</dt>
<dd>The combination of annotated content, one or more of: an annotation purpose, a flat text description, or a related annotation body.</dd>

<dt>annotated content</dt>
<dd>The range of text or other object which the annotation body is "about".<br><br>Note: In the [Web Annotations Data Model](https://www.w3.org/TR/annotation-model/#motivation-and-purpose), this is known as the **annotation target**. However, because the term "target" has a specific meaning in accessibility APIs as the end (and not beginning) of a relation, the term annotated content is preferred here, in order to avoid confusion with the actual directionality of `aria-details` in ARIA Annotation markup.

<dt>annotation body</dt>
<dd>Visible information within a page which is connected to annotated content for a related purpose.</dd>

<dt>annotation purpose</dt>
<dd>The role the annotation body plays in relation to the annotated content (e.g. comment, definition or footnote). 
</dd>
</dl>

# Implementations

## Chrome
- In IA2/ATK, exposes all of the roles via xml-roles object attribute
- Exposes aria-description if run with command line flag `--enable-features=AccessibilityExposeARIAAnnotations`
- ChromeVox supports a "move to previous/next same role"

## Firefox
- Exposes all of the roles via xml-roles
