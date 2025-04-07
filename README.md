# huh

A shell utility to explain command output using an LLM.

## Usage

Pipe the output of any command into `huh`. Optionally add `--verbose` for a more detailed explanation.

```bash
# Example: Get a concise summary of 'ls -lha' output
ls -lha | huh

# Example: Get a detailed explanation of 'ls -lha' output
ls -lha | huh --verbose

# Example: Explain the output of 'dig' concisely
dig google.com | huh

# Example: Explain the output of 'kubectl get pods' in detail
kubectl get pods -A | huh --verbose
```

`huh` will read the output provided via the pipe, send it to an LLM (configured for Google Gemini by default) with instructions for either a concise summary (default) or a detailed explanation (`--verbose`), and print the result to your terminal.

## Dependencies

- `curl`: Used to make API requests. Usually pre-installed on most systems.
- `jq`: Used to parse JSON responses from the API. You might need to install it:
    - Debian/Ubuntu: `sudo apt update && sudo apt install jq`
    - macOS (using Homebrew): `brew install jq`
    - Fedora: `sudo dnf install jq`
    - Others: Check your package manager or the [jq website](https://jqlang.github.io/jq/download/).

## Setup

1. Clone this repository.
2. Make the script executable: `chmod +x scripts/huh`
3. Configure your LLM API key:
    - Copy or rename `config/api_keys.conf.example` to `config/api_keys.conf` (if an example file exists) OR edit `config/api_keys.conf` directly.
    - Edit `config/api_keys.conf` and add your Google Gemini API key:
      ```bash
      # Get your API key from Google AI Studio: https://aistudio.google.com/app/apikey
      export GEMINI_API_KEY="YOUR_GEMINI_API_KEY_HERE"
      # Optional: Specify a model (defaults to gemini-1.5-flash)
      # export GEMINI_MODEL="gemini-1.5-pro"
      ```
    - *IMPORTANT*: Ensure `config/api_keys.conf` is listed in your `.gitignore` file (it is by default) to avoid committing secrets.
4. (Optional) Add the `scripts` directory to your PATH or create a symlink to `scripts/huh` in a directory already in your PATH (e.g., `/usr/local/bin` or `~/.local/bin`).

   Example symlink:
   ```bash
   # Ensure ~/.local/bin exists and is in your PATH
   mkdir -p ~/.local/bin
   # Create the symlink
   ln -s "$(pwd)/scripts/huh" ~/.local/bin/huh
   ```
   You might need to restart your shell or run `hash -r` for the new command to be recognized.

## Limitations

- **No Command Context:** The script only sees the *output* text piped into it. It doesn't know which command generated that output. The LLM explanation is based solely on the text provided. Providing the command context could lead to better explanations in some cases (see Future Ideas).
- **Input Size Limit:** To avoid accidentally sending huge amounts of data to the LLM API, the script currently truncates input after 1000 lines. This limit can be adjusted in the script if needed.
- **Prompt Reliance:** The quality and conciseness of the explanation depend heavily on the LLM's interpretation of the prompt ("concise summary", "detailed explanation", sentence limits). Results may vary.

## Future Ideas

- **Provide Command Context:** Allow optionally passing the command string as an argument for better LLM context, e.g., `ls -lha | huh --verbose "ls -lha"`. The script would need more robust argument parsing (`getopt`) and adjust the prompt.
- **Shell Integration (Alternative):** Implement the more complex shell hook method (see previous discussions) to allow the `command` then `huh` workflow, capturing output automatically. This requires user shell configuration changes.
- **Support other LLM backends:** Add configuration options for OpenAI, Claude, local models, etc.
- **More robust configuration:** Use environment variables more flexibly, potentially add command-line flags to override config.
- **Context awareness:** Optionally include previous commands or context in the prompt for more nuanced explanations (more relevant if command context is available).