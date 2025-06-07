# SFVRAG Chatbot Webpage

This repository hosts the static webpage for the SFVRAG Chatbot. The webpage provides a user interface for interacting with the chatbot.

## Overview

In essence, this repository provides a client-side, front-end interface (`index.html`) for an n8n-powered chatbot. Users interact with the webpage, send messages, and receive responses that are facilitated by an n8n webhook workflow acting as the backend. The webpage handles displaying the chat, managing chat history in the browser's local storage, and communicating with the n8n webhook.

## `index.html` Functionality

The `index.html` file is the core of the webpage and includes the following:

### HTML Structure
- **Header**: Contains the chatbot title ("SFVRAG-Chatbot"), navigation links (Startseite, Über uns, Kontakt), a help button, and a user profile icon. It also includes a mobile menu button for smaller screens.
- **Mobile Menu**: A hidden navigation menu (`<div id="mobileMenu">`) that becomes visible on mobile devices when the menu button is clicked.
- **Main Content**:
    - A welcome message and a brief instruction for the user.
    - A text area (`<textarea id="userInput">`) for users to type their messages.
    - A send button (`<button id="sendButton">`) to submit the message.
    - A response container (`<div id="responseContainer">`) which includes:
        - A "Verlauf" (History) section title.
        - A loading indicator (`<div id="loadingIndicator">`) shown while waiting for a response.
        - An error message display area (`<div id="errorMessage">`).
        - A container for the question and answer history (`<div id="qaHistoryContainer">`).
        - An initial message indicating an empty history (`<div id="emptyHistoryMessage">`).

### CSS Styling
- **Tailwind CSS**: The page heavily utilizes Tailwind CSS for its overall layout and styling. It's included via a CDN link.
- **Custom CSS**: Inline CSS within `<style>` tags is used for:
    - Loading dot animations (`loading-dots`).
    - Fade-in animation for new Q&A entries (`qa-entry`, `@keyframes fadeIn`).
    - Custom scrollbar styling for the chat history container (`#qaHistoryContainer::-webkit-scrollbar`).
    - Styling and transition animations for the mobile menu (`mobile-menu-enter`, `mobile-menu-leave`, etc.).
- **Google Fonts**: Imports "Inter" and "Noto Sans" fonts.

### JavaScript Logic
The core logic is contained within `<script>` tags:

- **Element Selection**: Variables are defined to hold references to key HTML elements (user input, send button, history container, loading indicator, error message, mobile menu elements).
- **Constants**:
    - `n8nWebhookUrl`: The URL for the n8n webhook that processes the chat messages.
    - `MAX_HISTORY_ITEMS`: Limits the number of chat history items stored in local storage to 10.
- **Mobile Menu Toggle (`toggleMobileMenu` function)**:
    - Handles the opening and closing of the mobile navigation menu.
    - Toggles CSS classes for animations and visibility.
    - Changes the menu button icon (hamburger to X and vice-versa).
    - Sets `aria-expanded` attribute for accessibility.
- **Event Listener for Mobile Menu**: Attaches `toggleMobileMenu` to the mobile menu button's click event.
- **Mobile Menu Link Click Handling**: Closes the mobile menu when a navigation link within it is clicked.
- **History Management**:
    - `loadHistory()`: Loads chat history from `localStorage` (key: `chatHistorySFV`) when the page loads. If no history exists, it shows the `emptyHistoryMessage`. Otherwise, it populates the `qaHistoryContainer` by calling `addQaToDom` for each stored item.
    - `saveHistory(question, answer)`: Saves a new question-answer pair to `localStorage`. It prepends the new item and ensures the history does not exceed `MAX_HISTORY_ITEMS`.
    - `addQaToDom(question, answer, isNew)`: Creates new DOM elements for a question and its corresponding answer and adds them to the `qaHistoryContainer`.
        - `isNew` flag controls whether to apply a fade-in animation.
        - Formats the answer by replacing newline characters (`\n`) with `<br>` tags and bolding text surrounded by double asterisks (`**text**`).
- **Message Sending (`sendMessage` async function)**:
    - Triggered by clicking the send button or pressing Enter (without Shift) in the input field.
    - Gets the user's question from `userInputElement`.
    - Shows a loading indicator and disables the send button.
    - Makes a `POST` request to the `n8nWebhookUrl` with the question in JSON format (`{ query: question }`).
    - **Response Handling**:
        - If the request is successful and the response contains a message, it calls `addQaToDom` to display the Q&A and `saveHistory` to store it. Clears the user input field.
        - If the HTTP response is not ok (e.g., 4xx, 5xx errors), it throws an error with status and details.
        - If the response structure is unexpected, it shows an error.
    - **Error Handling**: Catches errors during the fetch operation (network issues, HTTP errors, etc.) and displays an error message using `showError`.
    - **Finally Block**: Hides the loading indicator and re-enables the send button, regardless of success or failure.
- **Error Display (`showError(message)` function)**: Sets the text content of `errorMessageElement` and makes it visible.
- **Event Listeners**:
    - `sendButtonElement`: Listens for clicks to call `sendMessage`.
    - `userInputElement`: Listens for `keypress` events. If Enter is pressed without Shift, it prevents default form submission and calls `sendMessage`.
    - `DOMContentLoaded`: Calls `loadHistory` once the initial HTML document has been completely loaded and parsed.

### n8n Webhook Interaction
- When a user sends a message, the `sendMessage` function is triggered.
- It retrieves the user's input and makes an asynchronous `POST` request to the `n8nWebhookUrl` 
- The request body is a JSON object containing the user's query: `{ "query": "user's message" }`.
- The script then waits for a response from the n8n webhook.
- If the webhook call is successful and returns a JSON response with a `message` property, this message is considered the chatbot's answer.
- The answer is then displayed in the chat interface using the `addQaToDom` function and saved to the local storage history.
- Error handling is in place for network issues, HTTP errors from the webhook, or unexpected response formats.
