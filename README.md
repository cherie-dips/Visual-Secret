# Visual Secret

Interactive **threshold secret sharing** for images: split a secret picture into **n** grayscale “share” images so that any **k** of them recover the original exactly, while **k − 1** (or fewer) reveal no information about the secret. The demo runs entirely in the browser (no server).

**Live demo:** [https://cherie-dips.github.io/Visual-Secret/](https://cherie-dips.github.io/Visual-Secret/) (after you enable GitHub Pages as described below).

## How it works

This project implements **Shamir’s (k, n) secret sharing** [Shamir, 1979] on **each pixel** of a grayscale image.

1. **Secret**   The drawing or uploaded image is converted to **grayscale** (one byte per pixel, 0–255).

2. **Sharing (per pixel)**  
   For a pixel value **s**, the app builds a random polynomial of degree **k − 1** over **GF(2^8)** (256-element finite field):
   \[
   f(x) = s + a_1 x + \cdots + a_{k-1} x^{k-1}
   \]
   The coefficients \(a_1,\ldots,a_{k-1}\) are chosen uniformly at random in the field.  
   Share number **i** stores the **y**-value **f(i)** at **x = i** (for **i = 1 … n**). Each share is its own image: every pixel is one field element, shown as a gray level. Individual shares look like **high-entropy noise**.

3. **Field arithmetic**  
   All operations use **GF(2^8)** with the irreducible polynomial **x^8 + x^4 + x^3 + x + 1** (0x11b, same as AES). Multiplication, division, and Lagrange weights are implemented in JavaScript for this field.

4. **Reconstruction**  
   Given **k** distinct shares \((x_i, y_i)\), the secret at each pixel is recovered by **Lagrange interpolation** in GF(2^8), which uniquely determines the constant term **s = f(0)**. The demo uses the first **k** shares in the overlay order when more than **k** are present.

5. **Security intuition**  
   With **k − 1** shares, the secret pixel is **perfectly masked**: for every possible **s** there is exactly one choice of the missing coefficients that matches the observed shares. In the demo, if you overlay fewer than **k** shares, the preview shows **random noise** instead of a reconstruction, to illustrate that you are not seeing the secret.

### Name note: “visual” vs classical visual cryptography

**Classical visual cryptography** (e.g. Naor–Shamir) often means printing patterns on transparencies and **physically stacking** them so the eye sees the secret. This app instead shares **digital pixel values** with Shamir’s scheme and reconstructs by **computation**. The secret is still an image, but the math is threshold secret sharing in GF(256), not XOR-based visual layers.

## Using the app locally

Open `index.html` in a modern browser, or serve the folder with any static file server.

## GitHub Pages deployment

This repository includes a workflow that publishes the site with **GitHub Actions**.

1. Push the `main` branch (including `.github/workflows/deploy-pages.yml` and `index.html`).
2. In the repo on GitHub: **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to **GitHub Actions** (not “Deploy from a branch”) so the workflow is allowed to deploy.
4. After the workflow runs successfully, the site is available at  
   `https://<your-username>.github.io/Visual-Secret/`.

If you prefer not to use Actions, you can instead set Pages **Source** to **Deploy from a branch**, branch **main**, folder **/ (root)**; ensure `index.html` is at the repository root.

### Troubleshooting: `Get Pages site failed` / `HttpError: Not Found`

That means GitHub Pages is not enabled for this repo yet (or the source is not **GitHub Actions**). Do this once:

1. **Settings → Pages → Build and deployment**
2. Set **Source** to **GitHub Actions** (not “Deploy from a branch”).
3. Re-run the failed workflow (**Actions → workflow run → Re-run all jobs**).

The workflow also sets `enablement: true` on `configure-pages` so a later run can turn Pages on automatically when the token is allowed to do so; some orgs still require the manual step above.

## Repository layout

| File | Purpose |
|------|--------|
| `index.html` | Single-page demo (UI, GF(256) Shamir, generate/reconstruct) |
| `.github/workflows/deploy-pages.yml` | Builds and deploys static site to GitHub Pages |

## References

- A. Shamir, “How to share a secret,” *Communications of the ACM*, 1979.
