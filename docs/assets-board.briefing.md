# AssetsBoard Briefing

AssetsBoard is a Web Application for Private investors _to categorize and summarize investments_

## Main Features

1. `1_add_asset`

   - Add new asset with category, value, quantity and acquisition date

2. `2_update_asset`

   - Update asset with new value, quantity and acquisition date

3. `3_view_summary`
   - View categorized summaries

## Specifications

- **Interaction**: Web
- **Architecture**: Frontend_Backend
- **Database**: Relational
- **Authentication**: JWT
- **Integrations**: None
- **Presentation**: Responsive, Light and Dark mode, Lime and Cyan colors, Tomorrow and Fira Mono fonts

## Tech Stack

- **Frontend**: Angular
- **Backend**: Bun_Vanilla_TS
- **Database**: SQLite
- **E2E Testing**: Playwright
- **Code Quality**: Eslint
- **Styles**: PicoCSS


# Data Model for **AssetsBoard**

This document describes the data model for the **AssetsBoard** project. It covers the primary entities, their attributes, and their relationships.

## Main Entities

### User

- **Description**: Represents an investor using the system.
- **Attributes**:
  - **id**: integer - mandatory, unique (primary key, auto-increment)
  - **name**: string - mandatory
  - **email**: string - mandatory, unique
  - **password**: string - mandatory (hashed)
  - **created_at**: datetime - mandatory
  - **updated_at**: datetime - mandatory

### Asset

- **Description**: Represents an investment asset managed by a user.
- **Attributes**:
  - **id**: integer - mandatory, unique (primary key, auto-increment)
  - **category_id**: integer - mandatory (foreign key referencing Category)
  - **value**: decimal - mandatory
  - **quantity**: integer - mandatory
  - **acquisition_date**: date - mandatory
  - **user_id**: integer - mandatory (foreign key referencing User)
  - **created_at**: datetime - mandatory
  - **updated_at**: datetime - mandatory

### Category

- **Description**: Represents a classification for assets based on risk and liquidity.
- **Attributes**:
  - **id**: integer - mandatory, unique (primary key, auto-increment)
  - **name**: string - mandatory, unique
  - **risk**: string - mandatory
  - **liquidity**: string - mandatory
  - **created_at**: datetime - mandatory
  - **updated_at**: datetime - mandatory

## Relationships

- **User** **(one)** ||--o{ **Asset** **(many)**
  - Description: One user (investor) can own many assets.
- **Category** **(one)** ||--o{ **Asset** **(many)**
  - Description: One category can be associated with many assets.


## Metadata

- **Date**: 2025-02-12
- **Author**: [Alberto Basalo](https://albertobasalo.dev) , [albertobasalo@aicode.academy](albertobasalo@aicode.academy)
- **Company**: [AI code Academy](https://aicode.academy)


_End of Architecture Document for AssetsBoard_