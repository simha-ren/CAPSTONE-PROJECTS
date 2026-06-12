# Smart FAQ Bot

This project implements a simple, yet effective, Smart FAQ Bot using a combination of vectorization, FAISS for efficient similarity search, and the Groq API for generating concise answers based on retrieved FAQs.

## Table of Contents
- [Project Overview](#project-overview)
- [Features](#features)
- [Setup and Installation](#setup-and-installation)
- [How to Use](#how-to-use)
- [Customizing FAQs](#customizing-faqs)
- [Core Components](#core-components)

## Project Overview

This bot is designed to answer user queries by finding the most relevant questions and answers from a predefined list of FAQs. It leverages text vectorization to represent questions as numerical vectors, uses a FAISS index for fast nearest-neighbor search, and then utilizes the Groq API to generate a concise answer based on the retrieved relevant FAQs.

## Features

- **Semantic Search**: Understands the meaning of user queries to retrieve relevant FAQs.
- **Fast Retrieval**: Uses FAISS for high-performance similarity search.
- **Concise Answers**: Employs the Groq API to synthesize answers from the retrieved FAQs.
- **Offline Mode**: Provides a fallback answer if the Groq API is unavailable or the API key is not set.

## Setup and Installation

To get this project up and running, follow these steps:

1.  **Clone the Repository (if applicable)**:
    ```bash
    git clone <your-repo-link>
    cd <your-repo-name>
    ```

2.  **Install Dependencies**:
    The bot requires `faiss-cpu`, `groq`, `scikit-learn`, and `python-dotenv`. You can install them using pip:
    ```bash
    !pip install faiss-cpu
    !pip install groq
    !pip install scikit-learn
    !pip install python-dotenv
    ```
    (Note: These are already included as `!pip install` commands in the main notebook cell.)

3.  **Set up Groq API Key**:
    The bot uses the Groq API for generating answers. You'll need a Groq API key.

    -   **Option 1: Using a `.env` file**
        Create a file named `.env` in the root directory of your project and add your Groq API key to it:
        ```
        GROQ_API_KEY=your_groq_api_key_here
        ```
        The `dotenv` library will automatically load this key.

    -   **Option 2: Setting it directly in the code**
        You can set the environment variable directly in your Python code before initializing `Groq`:
        ```python
        import os
        os.environ["GROQ_API_KEY"] = "your_groq_api_key_here"
        # Or, if using Google Colab secrets:
        # from google.colab import userdata
        # os.environ["GROQ_API_KEY"] = userdata.get('GROQ_API_KEY')
        ```

## How to Use

Once the dependencies are installed and your Groq API key is set, you can run the bot:

1.  **Execute the main code cell** in your notebook. This will initialize the vectorizer, FAISS index, and load your FAQs.
2.  The bot will then prompt you with `You: `.
3.  Type your query and press Enter.
4.  The bot will retrieve relevant FAQs and generate an answer.
5.  Type `exit` to quit the bot.

Example Interaction:

```
Smart FAQ Bot. Type 'exit' to quit.

You: What time does the library close?

Bot: The library is open from 9 AM to 9 PM, Monday through Friday, and 10 AM to 5 PM on weekends.
Top scores: 0.987, 0.721

You: I want to return a book

Bot: To borrow a book, present your library card at the circulation desk.
Top scores: 0.850, 0.600

You: exit
```

## Customizing FAQs

You can easily customize the list of questions and answers by modifying the `FAQS` list in the main Python cell:

```python
FAQS = [
    "Q: What are the library hours? A: The library is open from 9 AM to 9 PM, Monday through Friday, and 10 AM to 5 PM on weekends.",
    "Q: How can I borrow a book? A: You can borrow books by presenting your library card at the circulation desk.",
    # Add or modify FAQs here
]
```
After changing the `FAQS` list, you must re-run the cell to re-index the new questions.

## Core Components

-   **`FAQS`**: A list of strings, where each string is a "Q: Question A: Answer" pair. These are the knowledge base for the bot.
-   **`HashingVectorizer`**: Used to convert the text questions into numerical feature vectors. `n_features=384` defines the dimensionality of these vectors. `norm=None` means no normalization is applied by the vectorizer itself, as custom normalization is applied later.
-   **`numpy.normalize`**: Applies L2 normalization to the feature vectors. This is crucial for cosine similarity-based search, as FAISS's `IndexFlatIP` (Inner Product) index becomes equivalent to cosine similarity when vectors are L2-normalized.
-   **`faiss.IndexFlatIP`**: A FAISS index that performs an inner product search. Since the vectors are L2-normalized, this effectively finds the questions most similar to the user's query (highest cosine similarity).
-   **`retrieve(query, top_k=2)`**: This function takes a user query, converts it to a vector, searches the FAISS index for the `top_k` most similar FAQ vectors, and returns the corresponding FAQ pairs along with their similarity scores.
-   **`generate_answer(query, matches)`**: This function takes the user's original query and the retrieved FAQ matches. It then uses the Groq API (if `GROQ_API_KEY` is set) to generate a concise answer, using the retrieved FAQs as context. If the API key is not set, it falls back to providing the answer from the top-scoring retrieved FAQ.
