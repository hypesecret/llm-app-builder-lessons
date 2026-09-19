| Model | Attempts | Successes | $/attempt | **$/success** | s/attempt |
|---|---|---|---|---|---|
| deepseek/deepseek-v4.1-flash | 14 | 13 | 0.0044 | **0.0047** | 22 |
| openai/gpt-5.6-luna | 14 | 11 | 0.0039 | **0.0049** | 6 |
| deepseek/deepseek-v4-pro-0813 | 14 | 11 | 0.0097 | **0.0124** | 43 |
| z-ai/glm-5.3 | 14 | 10 | 0.0164 | **0.0230** | 9 |
| qwen/qwen3.8-max-0902 | 6 | 6 | 0.0246 | **0.0246** | 52 |
| google/gemini-3.8-flash | 6 | 4 | 0.0195 | **0.0292** | 16 |
| anthropic/claude-haiku-4.5 | 12 | 10 | 0.0249 | **0.0299** | 15 |
| anthropic/claude-sonnet-5 | 19 | 17 | 0.0586 | **0.0655** | 21 |

| App | Model | Cost | Time | Output tok/s | Result |
|---|---|---|---|---|---|
| salon | anthropic/claude-sonnet-5 | $0.216 | 146 s | 136 | ✅ renders |
| salon | deepseek/deepseek-v4-pro-0813 | $0.032 | 442 s | 22 | ✅ renders |
| salon | z-ai/glm-5.3 | $0.039 | 33 s | 246 | ❌ compiles but blank page |
| salon | qwen/qwen3.8-max-0902 | $0.067 | 198 s | 55 | ✅ renders |
| salon | google/gemini-3.8-flash | $0.048 | 60 s | 203 | ✅ renders |
| salon | openai/gpt-5.6-luna | $0.010 | 35 s | 225 | ✅ renders |
| salon | openai/gpt-5.6-sol | $0.128 | 114 s | 105 | ✅ renders |
| boutique | anthropic/claude-sonnet-5 | $0.257 | 170 s | 141 | ❌ truncated by the output cap |
| boutique | deepseek/deepseek-v4-pro-0813 | $0.060 | 432 s | 47 | ❌ does not compile |
| boutique | z-ai/glm-5.3 | $0.050 | 39 s | 337 | ❌ does not compile |
| boutique | qwen/qwen3.8-max-0902 | $0.071 | 208 s | 56 | ✅ renders |
| boutique | google/gemini-3.8-flash | $0.069 | 84 s | 211 | ✅ renders |
| boutique | openai/gpt-5.6-luna | $0.013 | 61 s | 183 | ❌ does not compile |
