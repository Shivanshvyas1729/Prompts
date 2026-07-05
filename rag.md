<details>
<summary><strong>History-Aware Query Reformulation</strong></summary>

```python
contextualize_q_system_prompt = (
    "Given a chat history and the latest user question "
    "which might reference context in the chat history, "
    "formulate a standalone question that can be understood "
    "without the chat history. "
    "Do NOT answer the question, only rewrite it if necessary."
)
```

</details>

<details>
<summary><strong>Question-Answer Prompt</strong></summary>

```python
qa_system_prompt = (
    "You are a helpful AI assistant.\n\n"
    "Use the following retrieved context to answer the user's question.\n"
    "If you don't know the answer, simply say you don't know.\n\n"
    "Context:\n{context}"
)
```

</details>
