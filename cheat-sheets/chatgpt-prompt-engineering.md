# ChatGPT Prompt Engineering

## Role Priming

First prompt should look like this:

```
Ignore all previous instructions.

You are an expert <ROLE>. You've been <DOING THING> for 20 years.

Your task is now to <SPECIFIC AND DETAILED INSTRUCTIONS>.

Reply simply with "Ok" to acknowledge.
```

Source: [Underscore_](https://www.youtube.com/@Underscore_)'s [Prompt Engineering](https://www.youtube.com/watch?v=rnDbi8gti0U)

Examples: [ChatGPT Prompt Engineering Example: Role Priming](https://github.com/gnugat/knowledge/blob/master/cheat-sheets/chatgpt-prompt-engineering-examples.md#role-priming)

## Iterative Improvement

First generate criticisms:

```
Now,
  [as an expert <ROLE> who is known for their harsh yet constructive criticism,]
  provide me with brutally honest feedback about that <THING>
  and convince me why it's bad. Let's think step by step.
```

Then generate improvement suggestions:

```
Great points. Now suggest improvements to <THING>, based on your critics
  [, and convince me why it's better]. Let's think step by step.
```

Finally generate new improved version:

```
Great. Now rewrite <THING> and improve it based on your suggestions.
```

Source: [Goda Go](https://www.youtube.com/@godago)'s [Criticize Me Mode](https://www.youtube.com/watch?v=EYjG6i53-xk)
