# Evaluate Reverse Polish Notation

LeetCode 150.

## Problem model

A postfix expression supplies operands before the operator that consumes them.

Examples of clarified constraints:

- input tokens are valid;
- division by zero does not occur;
- integer division truncates toward zero;
- every operator has enough operands;
- the expression produces one final integer.

## Stack insight

Iterate left to right:

1. Parse and push numeric operands.
2. On an operator, pop the right operand first and the left operand second.
3. Evaluate `left operator right`.
4. Push the intermediate result for later operators.
5. The remaining stack value is the answer.

Operand order matters for subtraction and division:

```csharp
int rightOperand = stack.Pop();
int leftOperand = stack.Pop();
```

## Submitted solution

```csharp
public class Solution
{
    public int EvalRPN(string[] tokens)
    {
        Stack<int> stack = new Stack<int>();

        foreach (string token in tokens)
        {
            if (int.TryParse(token, out int operand))
            {
                stack.Push(operand);
                continue;
            }

            int rightOperand = stack.Pop();
            int leftOperand = stack.Pop();

            switch (token)
            {
                case "+":
                    stack.Push(leftOperand + rightOperand);
                    break;
                case "-":
                    stack.Push(leftOperand - rightOperand);
                    break;
                case "/":
                    stack.Push(leftOperand / rightOperand);
                    break;
                case "*":
                    stack.Push(leftOperand * rightOperand);
                    break;
            }
        }

        return stack.Pop();
    }
}
```

The implementation deliberately trusted the stated valid-input contract instead of adding unnecessary defensive branches.

## Complexity

```text
Time:  O(n)
Space: O(n)
```

## Production-oriented explicit stack uses

Explicit `Stack<T>` remains less common than `List<T>`, dictionaries and queues in CRUD-heavy services, but it is appropriate for:

- undo/redo command history;
- iterative depth-first traversal;
- expression and rule evaluation;
- parser state;
- reverse-order compensation actions;
- navigation/backtracking state.
