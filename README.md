# onQ CMS API Reference

The onQ CMS offers an API to allow third party services to interact with the CMS and players linked to the CMS. Typically, this is used to automate content uploading and playlist generation. onQ Digital also offer a plugin service to link the CMS to third party services with an API. Please talk to an onQ sales consultant if you wish to explore a plugin between the onQ CMS and a third party platform.

All onQ API requests are made over HTTP using a POST request to a specified endpoint at **https://onqcms.com/api/{api-endpoint}**. All requests must be authenticated using a generated API token linked to the account the request is for. A successful API request will receive a **200 Ok** response. A **404 Not Found** response is returned if a request is made to a non-existing endpoint. A **500 Internal Error** response will be returned if an API request with an error in the payload is received.

The curl examples below are all suitable for a bash shell. The syntax may vary for other shells.

## Authorization and Token Generation

All requests to the onQ CMS must be authenticated by a unique API token that can be generated for an onQCMS account.

### Token Generation

An admin user can create an API token by logging into the onQ CMS in their web browser (https://onqcms.com/). API keys can be managed by going to the Admin menu, and then opening the API tab. Clicking the Add API Key button will generate an API key for you. You can give the key a name for your reference under the APN Name field. API keys can also be deleted from this menu which invalidates that key for all future requests.

### Authentication

Requests to the OnQ CMS must be authenticated with a valid token generated in **1.1 Token Generation**. The token should be set in the **Authentication** header of the request. Any request without a valid token will receive a **403 Forbidden** response from the server. We use the bearer token format to send the token:

`curl -H "Authorization: bearer demo123"`

## API Endpoints

- [Content](content.md)
- [Player](player.md)
- [Playlist](playlist.md)
- [Report](report.md)
- [Category & Tag](category-tag.md)
- [Seedooh Integration](seedooh.md)
- [API Job (Async Polling)](api-job.md)
