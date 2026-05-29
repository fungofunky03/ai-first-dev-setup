---
title: "Extract component"
category: refactor
when_to_use: "When a file/function/component has grown beyond ~200 LOC and you can identify a self-contained subsection worth lifting out"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Mechanical refactor that pulls a subsection of a component or function into
its own module without changing behaviour. Models are good at this when
constrained to "lift, don't redesign", but will silently improve unrelated
code if not stopped — this prompt enforces equivalence and a small,
reviewable diff.

## Prompt

```
Extract <TARGET> from <SOURCE_FILE> into a new module at <NEW_PATH>. This
is a behaviour-preserving refactor.

Rules:
1. The extracted unit must be a single export (one function, one component,
   one class) plus any private helpers it owns.
2. Public API of the source file must not change. Any caller outside the
   source file should compile and run identically.
3. Do not rename anything. Do not change types beyond what is needed to
   make the extraction work (e.g. exposing a parameter that was
   previously a closure variable).
4. Do not fix unrelated issues you notice. List them at the end under
   "Out of scope" — do not include them in the diff.
5. Preserve all comments and JSDoc/TSDoc on the extracted unit.

Output:
- The new file at `<NEW_PATH>` in full.
- A diff of `<SOURCE_FILE>` showing the removal and the new import.
- A list of any callers (outside `<SOURCE_FILE>`) that need to be updated.
  If none, say so.
- "Out of scope" — issues you saw and deliberately did not fix.

Source file:
<paste file contents>
```

## Example

**Input:** `src/components/CheckoutForm.tsx` (300 LOC) where lines 85-180
implement an address autocomplete dropdown that has its own state and
keyboard handling.

`TARGET` = "the address autocomplete logic"
`SOURCE_FILE` = `src/components/CheckoutForm.tsx`
`NEW_PATH` = `src/components/AddressAutocomplete.tsx`

**Expected output structure:**

```tsx
// src/components/AddressAutocomplete.tsx (new file)
import { useState, useEffect } from "react";

type AddressSuggestion = { id: string; label: string };

export function AddressAutocomplete({
  value,
  onChange,
  onSelect,
}: {
  value: string;
  onChange: (v: string) => void;
  onSelect: (s: AddressSuggestion) => void;
}) {
  const [suggestions, setSuggestions] = useState<AddressSuggestion[]>([]);
  // ...lifted state, effects, and keyboard handlers...
  return (
    <div className="autocomplete">
      {/* ...lifted JSX... */}
    </div>
  );
}
```

```diff
// src/components/CheckoutForm.tsx
+import { AddressAutocomplete } from "./AddressAutocomplete";

 export function CheckoutForm() {
-  const [addressInput, setAddressInput] = useState("");
-  const [suggestions, setSuggestions] = useState<AddressSuggestion[]>([]);
-  // ... 90 lines of autocomplete logic ...
+  const [addressInput, setAddressInput] = useState("");

   return (
     <form>
-      <div className="autocomplete">
-        {/* ... 30 lines of autocomplete JSX ... */}
-      </div>
+      <AddressAutocomplete
+        value={addressInput}
+        onChange={setAddressInput}
+        onSelect={(s) => setSelectedAddress(s)}
+      />
       {/* ... rest of form ... */}
     </form>
   );
 }
```

> Callers outside `CheckoutForm.tsx` that need updating: None — autocomplete
> was previously inlined.
>
> **Out of scope:**
> - `CheckoutForm` mixes form state and API calls. Worth splitting later.
> - `AddressSuggestion` type is duplicated in `src/api/places.ts`; should
>   be unified in a `types/` directory.

## Personalisation tips

- **Different framework:** the rules are framework-agnostic. For Vue,
  React Native, Svelte, Solid, etc., the only change is "component" syntax.
  For non-component languages (Python, Go), reuse this for extracting a
  function or class into its own module.
- **Aggressive extraction:** if you want the model to *propose* an
  extraction point rather than executing one you have already chosen,
  start the prompt with "Identify the largest cohesive subsection of
  <SOURCE_FILE> that can be safely extracted, propose a name and path,
  then wait for confirmation before extracting."
- **Hard-mode equivalence proof:** append "After the refactor, list every
  observable behaviour of the original code and show how each is preserved
  in the new structure." Slower, much higher confidence.
