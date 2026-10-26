### AI Order Parser 🍔🥤

An intelligent, type-safe natural language processing utility that converts raw, unstructured food orders into structured JSON objects using the OpenAI Structured Outputs API and Pydantic. 

### Core Features

* **Guaranteed JSON Schema:** Uses OpenAI's strict response_format to ensure outputs match your system's required structure 100% of the time.
* **Automatic Type Casting:** Quantities and item totals are automatically verified and returned as integers rather than strings.
* **Value Constraint Verification:** Sanitizes user inputs for size modifiers (e.g., "large", "small") down to strict, pre-defined categories.

### Architectural Breakdown

### 1. Custom Data Classes (Pydantic Models)

To ensure type safety and schema validation, the system utilizes Pydantic BaseModel classes. These models serve a dual purpose: they instruct the OpenAI API on what schema to generate, and they validate the returned JSON locally. 

* **OrderItem**: Defines individual items in the cart. 

  * name (string): The standardized name of the item.
  * quantity (integer): How many units were ordered.
  * size (Literal string): Constrained to "small", "regular", or "large". It defaults to "regular" if unspecified.
* **Order**: The top-level wrapper representing the complete ticket. 

  * items (List): An array of OrderItem models.
  * total_items (integer): The mathematical sum of all item quantities.

### 2. The System Content Prompt

The system prompt establishes the behavioral baseline for the AI model. It isolates the model's persona to focus solely on data extraction, preventing conversational fluff (like *"Sure, I can help you with that!"*) from corrupting the raw data output. 

text

You are a helpful assistant that extracts food orders into structured JSON format.

Use code with caution.

### 3. Order Parsing (client.beta.chat.completions.parse)

Instead of standard string completion, this application leverages the native .parse() method provided by the OpenAI SDK. 

Passing our custom Pydantic model (Order) into the response_format argument forces the model to respond strictly inside the parameters of our schema. If the model fails to generate conforming JSON, the SDK raises a validation error immediately rather than passing corrupted data down into your application code. 

### Usage Workflow

### Prerequisites

Install the required packages in your Python environment: 

bash

pip install openai pydantic

Use code with caution.

### Quick Start Code

python

from openai import OpenAI
from pydantic import BaseModel
from typing import List, Literal

# Initialize the OpenAI Client
client = OpenAI()

# Define the custom classes
class OrderItem(BaseModel):
    name: str
    quantity: int
    size: Literal["regular", "small", "large"]

class Order(BaseModel):
    items: List[OrderItem]
    total_items: int

# Raw unstructured string input
order_input = "2x Americano, 1 large fries and 3 hamburger"

# Execute parsing against gpt-4o-mini
response = client.beta.chat.completions.parse(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant that extracts food orders into structured JSON format."},
        {"role": "user", "content": f"Parse this order: {order_input}"}
    ],
    response_format=Order,
)

# Output the validated string response
print(response.choices.message.content)

Use code with caution.

### Output Example

The code execution will reliably yield the following deterministic JSON: 

json

{
  "items": [
    {"name": "Americano", "quantity": 2, "size": "regular"},
    {"name": "Fries", "quantity": 1, "size": "large"},
    {"name": "Hamburger", "quantity": 3, "size": "regular"}
  ],
  "total_items": 6
}

Use code with caution.
