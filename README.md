# FrontierOn8GB: Running Llama-3-70B on an 8GB M1 MacBook Air

## The Premise
Conventional wisdom states that running a 70-Billion parameter frontier model requires 40GB+ of VRAM. We proved that wrong. Using macOS Unified Memory, aggressive SSD swap paging, and `llama.cpp`, we successfully ran **Meta-Llama-3-70B-Instruct** on an 8GB M1 MacBook Air without crashing the OS.

## The Evidence
- **Hardware:** MacBook Air (M1, 8GB Unified Memory, 256GB SSD)
- **Model:** `Meta-Llama-3-70B-Instruct-Q3_K_M.gguf` (34.3 GB file)
- **Memory Pressure:** Yellow (Stable for 3 hours and 50 minutes)
- **Active Swap Used:** ~11.78 GB (Dynamic OS Allocation peaked at ~30 GB)
- **Performance:** `[ Prompt: 0.5 t/s | Generation: 0.0 t/s ]`
- **Time to Generate:** ~230 minutes for a 150-token response.

## The Breakthrough: Defeating the Jetsam Panic
The critical failure mode we had to avoid was the macOS "Jetsam" kernel panic. By default, `llama.cpp` tries to offload the model to the Metal GPU, which instantly exhausts the 8GB of unified memory. 

By forcing `-ngl 0` (Number of GPU Layers = 0), we forced the CPU and SSD to handle the layer-wise streaming. macOS dynamically allocated up to 30 GB of swap space (proven via `sysctl vm.swapusage`) to keep the process alive, proving that local frontier AI is no longer gated behind $4,000 hardware.

## The Exact Command
```bash
caffeinate -dims llama-cli \
  -m ./llama3-70b/Meta-Llama-3-70B-Instruct-Q3_K_M.gguf \
  -ngl 0 \
  -t 4 \
  --no-warmup \
  -p "You are a hardware optimization expert. Explain how layer-wise model streaming allows a 70-billion parameter frontier model to run on a laptop with only 8GB of unified memory, and why this democratizes access to advanced AI for developers with constrained hardware." \
  -n 150 \
  -c 512

