

# PDF Inverter

This is a light-weight, client-side application for generating high-quality dark mode versions of PDF documents. This project overcomes inherent limitations of native PDF inversion e.g., failed inversion on transparent backgrounds, color-keying failures and improper inversion of rasterized text elements by rebuilding the document layer-by-layer using graphics blending techniques.

---

## Project Overview

This project uses **pdf-lib** to systematically decompose, reconstruct, and re-render each page, enforcing a full inversion without breaking anything.

* **Primary idea:** Three-layer page construction using `BlendMode.Difference`.
* **Stability Goal:** Reliable processing for documents that crash or produce errors in other client-side PDF tools.

---

## Technical Implementation: The Three-Layer Blend

The core of the inversion process is the strategic use of the **Difference Blend Mode** over a clean, forced background. Difference blending subtracts the darker color from the lighter color component, resulting in a perfect inversion when the target color is white:

`Color_Output = |Color_Top - Color_Base|`

### Layer Construction

| Layer No. | Layer Content                      | Blend Mode | Purpose                                                                                                                                                    |
| --------- | ---------------------------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1         | Solid White Rectangle (rgb(1,1,1)) | Normal     | **Base:** Provides a non-transparent white canvas. This is to solve inversion issues on transparent backgrounds.                                         |
| 2         | Original Cloned Page Content       | Normal     | **Content Projection:** Draws all original text, vectors, and images onto the controlled white base.                                                       |
| 3         | Solid White Rectangle (rgb(1,1,1)) | Difference | **Inversion Operator:** Applies the difference mask to Layer 1 and Layer 2, resulting black where the base was white and white where the content was black. |

---

## Stability Engineering: Atomic Page Cloning

PDF libraries like **pdf-lib** can throw runtime exceptions when processing complex or corrupted documents. To prevent crashes:

* **Avoided method (unstable):**

```javascript
// const embeddedPages = await newDoc.embedPages(srcDoc, pageIndices);
```

* **Implemented method (stable):**

```javascript
const embeddedPages = [];
for (const page of srcDoc.getPages()) {
    // Forces a clean object copy for each page, bypassing corruption.
    const embeddedPage = await newDoc.embedPage(page);
    embeddedPages.push(embeddedPage);
}
```

This isolates object reference issues to individual pages, ensuring that one corrupted page does not break the entire document.

---

## Technologies Used

| Technology        | Role                                    |
| ----------------- | --------------------------------------  |
| HTML5             | Structure and Canvas                    |
| Tailwind CSS      | Styling                                 |
| Lucide Icons      | Icons                                   |
| pdf-lib           | Core PDF processing and manipulation    |
| JavaScript (ES6+) | Client-side logic and async operations  |

---

## Installation and Usage

**Installation**

Since this is a single, self-contained HTML file, no server or build process is required.

```bash
# Clone the repository
git clone https://github.com/yourusername/true-pdf-inverter.git
cd true-pdf-inverter
```

**Usage**

1. Open `index.html` in any modern web browser.
2. Drag and drop your PDF onto the designated zone.
3. The application will process and invert the pages client-side.
4. View the live preview and click **Download PDF**.

**Hosted Version:**
You can try it directly online without cloning: [PDF Inverter Hosted](https://anshitsinha.github.io/pdf-inverter/)

---

## Contributing

We welcome contributions to enhance stability and performance. Please submit a **Pull Request** with clear commit messages and documentation for any changes.

---

## Contact

For questions or feedback, please [open an issue on GitHub](https://github.com/anshitsinha/pdf-inverter/issues).



---

Case study: [anshit.is-a.dev/works/pdf-inverter](https://anshit.is-a.dev/works/pdf-inverter) · Built by [Anshit Sinha](https://anshit.is-a.dev)
