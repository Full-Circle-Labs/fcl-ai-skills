# FCL AI Skills

Official [Claude Code](https://claude.ai/code) plugins from [Full Circle Labs](https://fullcirclelabs.bio) for AI-assisted sequencing workflows. Install these plugins to give Claude Code native knowledge of the FCL API to be able to place orders, check status, and retrieve sequence results directly from your terminal.

## Available plugins

| Plugin | Description |
|--------|-------------|
| [FCL API](plugins/interact-with-fcl-api/) | Place sequencing orders, check status, and retrieve FASTA results via the FCL API. Covers the full workflow: browse products, manage carts, place orders, poll status, and fetch assembled sequences. |

## Installation

### Claude Code 

1. Run `/plugin marketplace add Full-Circle-Labs/fcl-ai-skills`
2. Run `/plugin install interact-with-fcl-api@fullcirclelabs-fcl-ai-skills`

This installs all FCL plugins into your Claude Code environment.

## Configuration

After installing, set your FCL API key so Claude Code can authenticate requests:

1. Open your project settings:
   ```bash
   # In your project directory
   claude config
   ```
2. Add your API key under the `env` section of `.claude/settings.json`:
   ```json
   {
     "env": {
       "FCL_API_KEY": "your-api-key-here"
     }
   }
   ```
3. Restart your Claude Code session.

You can generate or find your API key [here](https://v1.fcl.bio/) under account settings. For more information, have a look at our [docs](https://docs.fcl.bio/docs/home/intro)

## Usage

Once installed and configured, Claude Code will automatically use the FCL skills when you ask it to interact with the FCL platform. For example:

- *"List available sequencing products and their prices"*
- *"Place a full-length plasmid sequencing order for these two samples"*
- *"Check the status of my latest order"*
- *"Download the FASTA sequences from order #12345"*

## Links

- [FCL API documentation](https://docs.fullcirclelabs.bio/docs/api)
- [Full Circle Labs](https://fullcirclelabs.bio)
- [Report issues](https://github.com/fullcirclelabs/fcl-ai-skills/issues)

## License

MIT — see [LICENSE](LICENSE).
