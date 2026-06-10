# Contributing

Thanks for your interest in improving the Automation Layer Adviser!

## Customising for your team

The decision framework is designed to be customizable. Here's how:

### 1. Update the decision logic

Edit `.github/workflows/automation-layer-adviser.yml` (lines 97–150) to modify:
- The five-step decision flow
- Test layer descriptions
- Team-specific rules
- Repo paths and component names

### 2. Add worked examples

If recommendations are off for your domain, add specific examples to the prompt (few-shot prompting). This significantly improves accuracy.

### 3. Adjust for specialized domains

The framework works well for general software engineering. For specialized domains (hardware, data pipelines, machine learning), you may need to:
- Redefine test layers
- Add domain-specific rules
- Update acceptance criteria evaluation

### 4. Testing changes

Test your customizations using the manual workflow trigger:
1. Go to **Actions → Automation Layer Adviser → Run workflow**
2. Fill in a real ticket from your project
3. Verify the recommendation in the Actions log
4. Iterate on the prompt until you're happy with the output

## Reporting issues

If you find bugs or have suggestions:
- Check existing issues first
- Include the ticket details that led to the unexpected recommendation
- Share the Claude output from the Actions log (if possible)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
