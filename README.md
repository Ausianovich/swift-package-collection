# Ausianovich Swift Package Collection

Swift Package Collection containing reusable packages for my iOS and server-side Swift projects.

## Collection URL

https://ausianovich.github.io/swift-package-collection/collection.json

## Important
Package collection generator: swift-package-collection-generator branch 5.10

# Quick start

## Updating and Signing the Package Collection

The package collection is generated from `packages.json`, signed with an Apple Swift Package signing certificate, validated, and then published through GitHub Pages.

### Prerequisites

The following tools must be available in `$PATH`:

```bash
package-collection-generate
package-collection-sign
package-collection-validate
```

The working generator version is:

```text
swift-package-collection-generator 5.10
```

Signing certificates and the private key are stored outside this repository:

```text
~/Documents/Developer/SwiftSigning/
├── swift-signing-key.pem
├── swift_package_certificate.cer
├── AppleWWDRCAG6.cer
└── AppleRootCA-G3.cer
```

Do not commit `swift-signing-key.pem` to Git.

---

## 1. Update `packages.json`

Add, remove, or update package repositories in:

```text
packages.json
```

Example:

```json
{
  "name": "Ausianovich Swift Packages",
  "overview": "Reusable Swift packages",
  "packages": [
    {
      "url": "https://github.com/Ausianovich/SomePackage.git"
    }
  ]
}
```

---

## 2. Generate the unsigned collection

Run from the repository root:

```bash

GITHUB_TOKEN="$(security find-generic-password \
  -a "$USER" \
  -s "package-collection-generator.github.com" \
  -w)"

package-collection-generate \
  packages.json \
  collection-unsigned.json \
  --auth-token "github:github.com:${GITHUB_TOKEN}" \
  --pretty-printed

unset GITHUB_TOKEN
```

This reads the package repositories and generates metadata including versions, products, targets, and Swift tools versions.

---

## 3. Sign the collection

Sign the generated collection using the private key and the full Apple certificate chain:

```bash
package-collection-sign \
  collection-unsigned.json \
  collection.json \
  ~/Documents/Developer/SwiftSigning/swift-signing-key.pem \
  ~/Documents/Developer/SwiftSigning/swift_package_certificate.cer \
  ~/Documents/Developer/SwiftSigning/AppleWWDRCAG6.cer \
  ~/Documents/Developer/SwiftSigning/AppleRootCA-G3.cer

rm collection-unsigned.json
```

Certificate order is important:

```text
Private key
    ↓
Swift Package Certificate
    ↓
Apple Worldwide Developer Relations CA G6
    ↓
Apple Root CA - G3
```

The resulting `collection.json` is the signed file published through GitHub Pages.

---

## 4. Validate the signed collection

Run:

```bash
package-collection-validate collection.json
```

The collection should pass validation before committing it.

---

## 5. Review changes

Check what changed:

```bash
git diff
```

Optionally inspect the generated collection:

```bash
cat collection.json
```

or:

```bash
python3 -m json.tool collection.json
```

---

## 6. Commit and push

```bash
git add packages.json collection-unsigned.json collection.json
git commit -m "Update package collection"
git push
```

GitHub Pages will publish the new version automatically from the `main` branch.

---

## Published Collection URL

```text
https://ausianovich.github.io/swift-package-collection/collection.json
```

This URL can be added to Xcode as a Package Collection.

---

## Full Update Workflow

For normal updates, the entire workflow is:

```bash
package-collection-generate \
  packages.json \
  collection-unsigned.json

package-collection-sign \
  collection-unsigned.json \
  collection.json \
  ~/Documents/Developer/SwiftSigning/swift-signing-key.pem \
  ~/Documents/Developer/SwiftSigning/swift_package_certificate.cer \
  ~/Documents/Developer/SwiftSigning/AppleWWDRCAG6.cer \
  ~/Documents/Developer/SwiftSigning/AppleRootCA-G3.cer

package-collection-validate collection.json

git diff

git add packages.json collection-unsigned.json collection.json
git commit -m "Update package collection"
git push
```

## Security

Never commit private signing keys.

Recommended `.gitignore` entries:

```gitignore
*.pem
*.p12
*.p8
*.csr
*.certSigningRequest
```

The Apple `.cer` files are public certificates, but the private key must remain outside the repository.
