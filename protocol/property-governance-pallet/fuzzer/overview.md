# Overview

The **Fuzzer** in the `pallet-property-governance` module is a **testing utility** designed to stress-test the governance logic with **randomised, yet structurally valid input data**.

Fuzz testing (or fuzzing) is critical in governance because decisions based on invalid or edge-case proposals could corrupt chain state, cause denial-of-service conditions, or allow unauthorised actions.
