# Certifly - Web-based Certificate Generation Tool

Certifly is an open-source web-based certificate generation tool written in Svelte by [Kathmandu university Open Source Commmunity (KUOSC)](https://kucc.ku.edu.np/kuosc/). It simplifies the process of creating certificates by allowing users to provide a template image and a CSV file containing recipient information.

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Easy Certificate Generation**: Generate customized certificates by providing a template image and recipient data in CSV format.

- **Template Customization**: Customize certificate appearance using template images with placeholders.

- **Supports Various Formats**: Certifly supports PNG and JPEG/JPG image formats for templates and CSV data.

- **Bulk Export**: Export generated certificates in bulk as a zip file.

## Getting Started

### Prerequisites

Before you begin, ensure you have the following installed:

- [Node.js](https://nodejs.org/)
- [pnpm](https://pnpm.io/) (Performant NPM)

### Installation

1. Clone the Certifly repository to your local machine:

   ```bash
   git clone git@github.com:kuosc2005/certifly.git
   cd certifly
   ```

2. Install the project dependencies:

   ```bash
   pnpm install
   ```

## Usage

1. **Prepare your template image:**

   Create a template image (e.g., `certificate_template.png`) with placeholders where you want recipient information to appear.

2. **Prepare your CSV data file:**

   Create a CSV file (e.g., `recipients.csv`). For example:<br>

   ```csv
   name,email,course,date
   John Doe,johndoe@example.com,Introduction to Certificates,2023-09-17
   Jane Smith,janesmith@example.com,Advanced Certificates,2023-09-18
   ```

3. **Start the Certifly application**

   ```bash
   pnpm dev
   ```

4. **Use the web interface:**

- Open your web browser and navigate to the Certifly application (typically at http://localhost:5173).

- Use the web interface to upload your template image and CSV data files. Make sure to specify the value in the template that match the columns in your CSV file.

- After uploading the necessary files, click the "Generate" button.

- Download the certificates by clicking on "Download" button.

## Contributing to Certifly

Please go through [CONTRIBUTING.md](https://github.com/kuosc2005/certifly/blob/main/CONTRIBUTING.md)

## License

Certifly is licensed under the [MIT License](https://opensource.org/license/mit/). Feel free to use, modify, and distribute it in your projects.
