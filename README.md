# Chat Popup Component
This is a simple chat popup component built with Vue 3 that allows users to interact with predefined options or ask custom questions. The component simulates a server response for user interactions.

## Features
Toggle Chat: Users can open and close the chat popup by clicking on the chat button.
Predefined Options: Users can choose from predefined options to get responses.
Custom Questions: Users can type and submit their own questions.
Typing Indicator: Displays a typing indicator when the bot is generating a response.
Scroll to Bottom: Automatically scrolls to the latest message when a new message is added.
Animations: Smooth animations for the chat popup and typing indicator.
Installation

## To integrate this chat popup component into your project, follow these steps:

Clone the Repository
bash
git clone https://github.com/PolinaSukhorukova/chat.git
cd chat

Install Dependencies
bash
npm install

Serve the Project
bash
npm run dev

Build for Production
bash
npm run build

## How to Use
Import the Component

In your Vue project, import the chat popup component and include it in your desired view.

<template>
  <div>
    <ChatPopup />
  </div>
</template>

<script setup>
import ChatPopup from './components/ChatPopup.vue';
</script>


The component comes with scoped styles to ensure it does not interfere with other styles in your project. You can customize the styles by editing the component's <style scoped> section.

## Customization
You can customize the component further by editing the ChatPopup.vue file:

License
This project is licensed under the MIT License.
