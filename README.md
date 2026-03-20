# Diceblox Provably Fair Audit Toolkit 🎲📉
https://waveyyfreshh.github.io/diceblox-pf-audit/

An independent, open-source mathematical audit and simulation engine designed to test, verify, and expose the statistical deviations within the Diceblox Blackjack algorithm.

This repository contains a standalone, client-side toolkit (`index.html`) that simulates the exact game logic used by Diceblox, proving the existence of a hardcoded 22% house override that bypasses their Provably Fair cryptographic generation.

## 📑 Table of Contents
- [Abstract & Findings](#abstract--findings)
- [The Exploit: Cryptography vs. Game Logic](#the-exploit-cryptography-vs-game-logic)
- [The 22% Override Code](#the-22-override-code)
- [How the Simulator Works](#how-the-simulator-works)
- [How to Cross-Check & Verify This Tool](#how-to-cross-check--verify-this-tool)
- [Usage Instructions](#usage-instructions)

---

## 🔬 Abstract & Findings
Online casinos utilize **Provably Fair (PF)** systems to assure players that outcomes are not manipulated. A standard PF Blackjack game generates a secure random hash, uses that hash to shuffle a deck, and draws cards. 

Our audit reveals that while Diceblox uses a secure Verifiable Random Function (VRF) to generate a secure hash, **their JavaScript game engine actively intercepts the hash output.** Exactly 22% of the time, the fair draw is aborted, and the dealer is artificially injected with a 19, 20, or Blackjack.

This override skyrockets the True House Edge from an industry-standard ~1.5% to **over 11%** (assuming Perfect Basic Strategy).

---

## 🕵️‍♂️ The Exploit: Cryptography vs. Game Logic
Platform operators often defend their systems by citing the strength of their cryptography (VRFs, Public Keys, Nonces). This is a known misdirection technique. A legitimate PF system requires two unbroken steps:
1. **Cryptography:** Generating a secure, immutable `Random Hash`.
2. **Game Logic:** Uniformly translating that `Random Hash` into a game outcome.

Diceblox passes Step 1 but fails Step 2. Rolling a mathematically perfect, cryptographically secure pair of dice is useless if the house rules dictate that landing on specific numbers automatically grants the dealer a winning hand. 

---

## 💻 The 22% Override Code
By inspecting the Diceblox web client (or viewing their official CodePen documentation), you can find the `generateRandomPair` function. 

This function takes the verified Random Hash and converts it into a "Ticket" (a float from 0 to 100). If the ticket rolls under 21, the code forces the dealer to receive premium cards, completely ignoring the natural deck shuffle for those indices.

```javascript
function generateRandomPair(chance) {
    const ticket = chance.floating({ min: 0, max: 100 });

    // The Override
    if (ticket < 3) return pickPair(11, 10);  // 3% prob: Forces Blackjack
    if (ticket < 9) return pickPair(10, 10);  // 6% prob: Forces Hard 20
    if (ticket < 15) return pickPair(9, 10);  // 6% prob: Forces Hard 19
    if (ticket < 21) return pickPair(8, 11);  // 6% prob: Forces Soft 19

    return null; // The remaining 79% of the time, the draw is fair
}
