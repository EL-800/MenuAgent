Here is the detailed **README.md** for the workflow, translated into English. It explains how the flow works, the services involved, and how to configure it based on the provided JSON file.

---

# 🍽️ WhatsApp Menu Agent (n8n Workflow)

This **n8n** workflow automates the creation and formatting of a daily restaurant menu. It allows you to receive dish details via **WhatsApp** (either by typing the text or sending a photo of a whiteboard/physical menu), processes the information using Artificial Intelligence, and returns a perfectly formatted **PDF** file based on a Google Docs template.

## 📋 Workflow Description

The system operates as follows:

1.  **Reception:** The user sends a message to the configured WhatsApp Business account.
2.  **Notification:** The bot responds immediately with a confirmation message: *"Vale, hare el menú..."* (Ok, I'll make the menu, this might take 1-2 minutes).
3.  **Input Detection:**
    *   **If it's an Image:** The workflow downloads the image and uses **Google Gemini Vision** to perform OCR and extract the text of the dishes.
    *   **If it's Text:** It uses the text directly from the message body.
4.  **AI Processing:** An AI Agent (Google Gemini) analyzes the text and structures it into a strict JSON format, categorizing dishes into:
    *   *Primer Tiempo* (First Course/Soups)
    *   *Segundo Tiempo* (Second Course/Sides)
    *   *Tercer Tiempo* (Third Course/Main Dishes)
    *   *Note: The AI is also instructed to abbreviate words like "salsa" to "s." to save space.*
5.  **Document Generation:**
    *   Copies an existing **Google Docs Template**.
    *   Replaces specific placeholders (e.g., `[Primer tiempo1]`) with the data structured by the AI.
    *   Exports the modified document to **PDF** format.
6.  **Delivery:** Uploads the PDF to WhatsApp servers and sends it back to the original user.

## 🛠️ Nodes and Services Used

### Required Services (Credentials)
*   **WhatsApp Cloud API:** To receive messages and send the resulting PDF.
*   **Google Gemini (PaLM) API:** For image analysis (Vision) and natural language processing (Chat).
*   **Google Drive & Google Docs:** To manage the template and file generation.

### Key Nodes Explanation
*   **WhatsApp Trigger:** Starts the workflow when a message is received.
*   **If (Conditional):** Checks if the message contains an image (`messages[0].image`).
*   **Analyze an image (Google Gemini):** If an image is present, it prompts the model: *"Obtén cuales platillos contiene la imagen"* (Get the dishes contained in the image).
*   **AI Agent + Structured Output Parser:** The "brain" of the workflow. It takes the raw text and forces a JSON output structured by courses.
*   **Google Drive (Copy):** Duplicates the master template to create today's menu file (e.g., `RinconDelSazonMenuDelDia3TT[DATE]`).
*   **Google Docs (Update):** Searches for placeholders in the new file and replaces them with the AI content.
*   **Google Drive (Download):** Downloads the Google Doc as a PDF binary.
*   **WhatsApp (Upload & Send):** Uploads the binary and sends it as a document attachment.

## ⚙️ Configuration and Prerequisites

To make this workflow run on your n8n instance, you need to configure the following:

### 1. Google Docs Template
You must have a Google Doc to serve as the base template. The workflow looks for a File ID `1APuxiYPXnpdL3hBa_FMs_MAPUwibk65nJnVCequdRQE` (you must replace this with your own file ID).

Inside your Google Doc, you must include the exact text placeholders below so the **Update a document** node can find them:
*   `[Primer tiempo1]`
*   `[Primer tiempo2]`
*   `[Segundo tiempo1]`
*   `[Segundo tiempo2]`
*   `[Tercer tiempo1]`
*   `[Tercer tiempo2]`
*   `[Tercer tiempo3]`

### 2. Node Configuration
After importing the JSON, check these specific nodes:

*   **Google Drive (Copy file):**
    *   Change the `File ID` to your master template's ID in Google Drive.
    *   Ensure the destination folder is correct (if applicable).
*   **AI Agent:**
    *   Review the `System Message` if you wish to change the abbreviation rules or the number of dishes per course.
*   **WhatsApp (Send Message):**
    *   Ensure the `Phone Number ID` matches your Meta for Developers account.

## ⚠️ Error Handling

The workflow includes basic error branches:
*   **Recognition Failed:** If the AI cannot structure the menu, it sends a message: *"No se logro reconocer el menú..."* (The menu could not be recognized...).
*   **Token Expired:** If the Google Drive copy operation fails, it sends an alert to a specific support number (`+52 1 55...`) stating: *"El token de Google drive expiro..."* (The Google Drive token expired...).

## 🚀 How to Use

1.  Open the WhatsApp chat with your bot.
2.  Type out today's menu:
    > "Lentil soup, Chicken broth. Red rice, Salad. Breaded chicken, Green enchiladas, Roast beef."
3.  **OR** take a photo of your kitchen's whiteboard and send it.
4.  Wait a few seconds, and you will receive a professionally formatted PDF ready to forward to your customers.
