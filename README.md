# AI-Powered Smart Incident Resolution API

A Mule 4 REST API for logging support incidents and resolving them faster with AI. It stores incidents in MySQL, processes them asynchronously through VM queues with retry handling, finds similar past incidents, and generates summaries using the Google Gemini API.

Built with Anypoint Studio and deployed on CloudHub.

## Features

- Create and fetch incidents through a RAML-designed REST API (APIkit)
- Upload logs or screenshots for an incident
- AI-generated incident summaries using the Gemini API
- Find similar past incidents using embeddings
- Asynchronous processing with VM queues
- Automatic retries (up to 3 attempts) and a dead-letter queue for failed messages
- Email notifications over SMTP
- Client ID enforcement (`client_id` and `client_secret` headers)
- Error handling, logging, and MUnit test cases

## Tech Stack

| Area | Tools |
|---|---|
| Runtime | Mule 4.11 (Enterprise), Java 17 |
| API design | RAML 1.0, APIkit |
| Transformations | DataWeave 2.0 |
| Database | MySQL |
| Messaging | Mule VM connector (incident, AI, email, retry, and dead-letter queues) |
| AI | Google Gemini API |
| Notifications | SMTP |
| Testing | MUnit |
| Build and deploy | Maven, Anypoint Platform (CloudHub) |

## API Endpoints

Base path: `/api`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/incidents` | Create a new incident |
| GET | `/incidents/{incidentId}` | Get incident details |
| POST | `/incidents/{incidentId}/logs` | Upload a log or screenshot (multipart/form-data) |
| GET | `/incidents/{incidentId}/similar` | Find semantically similar incidents |
| POST | `/incidents/{incidentId}/summarize` | Generate an AI summary for the incident and its logs |
| POST | `/incidents/retry` | Retry processing of failed incidents |

Interactive API documentation is served at `/console/`.

### Example: create an incident

```http
POST /api/incidents
Content-Type: application/json
client_id: <your-client-id>
client_secret: <your-client-secret>

{
  "customerId": "CUST1001",
  "title": "Database connection timeout",
  "description": "The application cannot connect to the database since 10 AM.",
  "severity": "HIGH",
  "category": "DATABASE",
  "contactEmail": "user@example.com"
}
```

Allowed values: `severity` is `LOW`, `MEDIUM`, `HIGH`, or `CRITICAL`; `category` is `NETWORK`, `DATABASE`, `APPLICATION`, `HARDWARE`, or `SECURITY`.

## Architecture

```
Client -> HTTP Listener -> APIkit Router -> Incident flows
                                              |
                          +-------------------+-------------------+
                          |                   |                   |
                       MySQL             VM queues            Gemini API
                                 (incident, AI, email, retry)  (summary, embeddings)
                                              |
                                  retry (up to 3) -> dead-letter queue
```

## Project Structure

```
src/main/mule/
  smart-incident-api.xml   # APIkit router and endpoint flows
  database-flows.xml       # MySQL operations
  vm-queues.xml            # Queue publishers, consumers, retry and dead-letter handling
src/main/resources/
  api/                     # RAML spec, types, traits, examples
  config.example.yaml      # Sample configuration (placeholders only)
src/test/munit/            # MUnit test cases
```

## Configuration

Real credentials are not stored in this repository. To run the project:

1. Copy `src/main/resources/config.example.yaml` to `src/main/resources/config.yaml`.
2. Fill in your own database, Gemini API, and SMTP values.
3. `config.yaml` is listed in `.gitignore` and must never be committed.

When deploying to CloudHub, set these values as properties in Runtime Manager (or use secure properties) instead of shipping them in the application.

## Run Locally

1. Install Anypoint Studio (Mule 4.11 runtime, Java 17).
2. Import the project: File > Import > Anypoint Studio > Packaged mule application, or open the project folder directly.
3. Create `config.yaml` as described above.
4. Create the `incident_db` MySQL database and required tables.
5. Right-click the project and choose Run As > Mule Application.
6. Open `http://localhost:8081/console/` (or your configured listener port) or call the endpoints with Postman.

## Testing

Run the MUnit tests from Anypoint Studio (right-click the project > MUnit > Run MUnit suite). Tests cover incident creation, similar-incident lookup, and AI summary success paths.

## Deployment

The application is deployed on CloudHub through Anypoint Platform (Runtime Manager). Set the configuration values as properties there before starting the app.

## Author

Sri Charan Rao Hibare
[LinkedIn](https://www.linkedin.com/in/sri-charan-rao-hibare) | [GitHub](https://github.com/SriCharan-23)


## CloudHub Deplyed Link :- https://smart-incident-resolution-app-o4ov3e.5sc6y6-3.usa-e2.cloudhub.io/
