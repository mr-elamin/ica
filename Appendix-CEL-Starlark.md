# Appendix: Logic Engines - CEL vs. Starlark

When working with Istio and Envoy, you will encounter two "Mini-Languages" used for logic: **CEL** and **Starlark**. Here is how to tell them apart and when to use each.

---

## 1. CEL (Common Expression Language)
**CEL** is what you use in Istio's `Telemetry` and `AuthorizationPolicy` resources.

*   **What it is:** A language designed to be "Small, Fast, and Safe." It was created by Google to provide a way to write expressions that can be evaluated in microseconds without ever crashing or hanging.
*   **Where it's used in Istio:**
    *   **Telemetry API:** In the `value:` field of `tagOverrides` (e.g., `value: "request.host"`).
    *   **Authorization Policy:** For custom conditions (e.g., `when: - key: request.auth.claims[iss] values: ["https://auth.com"]`).
    *   **Wasm Filters:** Many high-performance filters use CEL for configuration.
*   **Key Characteristics:**
    *   **Non-Turing Complete:** You cannot write infinite loops. This is a safety feature for sidecars.
    *   **Syntax:** Looks like C++/Java/Go logic. (e.g., `request.host.endsWith('.com')`).
    *   **Performance:** It is compiled into a bytecode that runs almost as fast as native code inside the proxy.

---

## 2. Starlark
**Starlark** is likely what you saw in the GitHub source code.

*   **What it is:** A dialect of **Python**. It was originally called *Skylark* and is the language used by **Bazel** (the build tool for Envoy and Istio).
*   **Where it's used:**
    *   **Build Infrastructure:** If you look at `BUILD` or `.bzl` files in the Istio/Envoy repos, that is Starlark.
    *   **Envoy Extensions:** Some advanced Envoy filters allow you to write logic in Starlark.
*   **Key Characteristics:**
    *   **Deterministic:** It is designed so that the same code always produces the same result (critical for builds).
    *   **Familiar:** If you know Python, you know Starlark.
    *   **Restricted:** It removes features like `class`, `yield`, and "imports" that could make builds non-reproducible or unsafe.

---

## 3. Comparison Table

| Feature | CEL | Starlark |
| :--- | :--- | :--- |
| **Primary Goal** | Expression Evaluation (Runtime) | Configuration/Build (Build-time/Complex) |
| **Syntax** | C/Go-like expressions | Python-lite |
| **Complexity** | Single-line logic | Multi-line scripts, functions |
| **Safety** | Guaranteed to finish (Fixed time) | Safe but allows more logic |
| **Istio Context** | `Telemetry` YAML `value:` | `BUILD` files in GitHub |

## Summary
*   Use **CEL** when you are **operating** Istio (editing YAMLs to add labels or check headers).
*   You will see **Starlark** when you are **developing** or building Istio/Envoy from source.

---
**[<< Chapter 15: Prometheus Metrics](15.istio-prometheus-metrics.md)** | **[Chapter 15.1: Custom Metrics >>](15.1-custom-metrics.md)**
