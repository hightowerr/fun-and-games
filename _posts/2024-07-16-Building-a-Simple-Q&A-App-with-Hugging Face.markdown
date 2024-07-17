---
layout: posts
title:  "Evaluating the UX Design of the Fantasy Premier League App"
date:   2024-07-09 12:30:00 +0100
categories: product, UX
author: Olayinka Ola
---

In this post, we'll explore how to create a straightforward question-and-answer user interface using Streamlit, LangChain, and Hugging Face. This application will allow users to ask questions and receive responses, similar to ChatGPT but through a different interface. We'll leverage Hugging Face's Transformers library for pre-trained models and tools to build our app quickly. The integration between Hugging Face and Streamlit simplifies this process significantly. Additionally, we'll use LangChain, a framework that facilitates the creation of Python applications and makes working with natural language data more manageable.

**Creating the App**
Setting Up the Environment
1. Create a free Hugging Face account.
2. Create a new space:
    - Name your space
    - Select Streamlit as the SDK
    - Choose free CPU space hardware
    - Set your space to public (recommended)

3. Add a requirements.txt file containing
    ```python
    langchain==0.2.5
    openai==1.35.3
    streamlit==1.36.0
    langchain-openai==0.1.9
    ```
4. Create an app.py file for your application's functionality.
    1. Imports necessary libraries

        ```python
        import streamlit as st
        from langchain_openai import OpenAI

        # Function to return the response
        def load_answer(question):
            llm = OpenAI(model_name="gpt-3.5-turbo-instruct", temperature=0)
            answer = llm.invoke(question)
            return answer
        ```
    2. Defines a function to load answers using the OpenAI model
        ```python
        # Function to return the response
        def load_answer(question):
            llm = OpenAI(model_name="gpt-3.5-turbo-instruct", temperature=0)
            answer = llm.invoke(question)
            return answer
        ```
    3. Sets up the Streamlit UI
        ```python
        # App UI starts here
        st.set_page_config(page_title="LangChain Demo", page_icon=":robot:")
        st.header("LangChain Demo")
        ```
    4. Creates input fields for user questions
        ```python
        # Gets the user input
        def get_text():
            input_text = st.text_input("User: ", key="input")
            return input_text

        # Get user input
        user_input = get_text()

        # Get response
        response = load_answer(user_input)
        ```
    5. Generates responses when the user clicks the "Generate" button
        ```python
        # Add button to generate response
        submit = st.button('Generate')

        # If generate button is clicked
        if submit:
            st.subheader("Answer:")
            st.write(response)
        ```

**Demonstration**

When you run the app, you'll see a simple interface where you can enter your question. After clicking the "Generate" button, the app will provide a response to your query. The app uses the GPT-3.5-turbo-instruct model with a temperature of 0 to generate direct and focused answers.

<iframe
	src="https://hightowerr-llmsintro.hf.space"
	frameborder="0"
	width="850"
	height="450"
></iframe>

**Conclusions**

In this post, we've walked through the process of building an interactive question-answering app using Streamlit and Hugging Face. We've explained each step of the code and demonstrated how to create a simple yet powerful natural language processing application. This project showcases the ease of integrating advanced AI models into web applications using modern tools and frameworks.