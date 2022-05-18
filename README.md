# Using Remote Fields to Integrate Content With GraphCMS

**Remote fields** allow you to add content from external systems and sources to GraphCMS via the API without having to migrate the content itself.

There may be many reasons why moving content to the CMS is not possible or desirable, such as:
* A different system of record is used to store content. Examples include content from a different vendor — for example, Github or IMDBT — as well as e-commerce or product data, such as price or availability.
* The content is in a legacy system for which the migration process is difficult.

With remote fields, you can create a single GraphQL API with content from multiple external sources, which provides more flexibility for frontend developers and downstream applications.

## Introducing Key Concepts

Before you get started with integrating data, learn more about some important concepts related to this process.

<dl><dt>Remote source
  </dt><dd>A system or product storing content that should be merged with content in GraphCMS. The content is fetched through a RESTful or GraphQL API. A remote source may include different types of content, for example, an e-commerce platform may have products, categories, and prices. Each remote source has a unique (base) URL. Examples of remotes sources are Github, Shopify, Hasura, custom backend applications, and more.</dd>

<dl><dt>Custom type
   </dt><dd>A <a href="https://graphql.org/learn/schema/" target="_blank">GraphQL type</a> that is used for content coming from a remote source. Custom types are combined with the auto-generated GraphCMS types to create a single schema for content in GraphCMS and in the remote source. For RESTful remote sources, you need to define custom types explicitly using <a href="https://www.prisma.io/blog/graphql-sdl-schema-definition-language-6755bcb9ce51" target="_blank">SDL</a> for all URL paths that will be queried in remote fields. For GraphQL remote sources, custom types are auto-generated through <a href="https://graphql.org/learn/introspection/" target="_blank">introspection</a> on the remote source.</dd>
  
<dl><dt>Custom input type
  </dt><dd>A custom <a href="https://graphql.org/learn/schema/#input-types" target="_blank">GraphQL input type</a> that is used to define input parameters for queries to remote sources.</dd>
  
<dl><dt>Remote field
</dt><dd>A field in a GraphCMS model designed to connect specific remote data to an entry in that model. Remote fields are always related to a single remote source and custom type. RESTful remote fields are configured with a path to a specific endpoint in the remote source. Examples of remote fields are user details on Github or price and availability information in Shopify.</dd>
  
## Configuring Remote Fields
  
To enable the integration of content from an external source to GraphCMS through remote fields, complete the following steps:
1. [Add a remote source](#Step-1-Add-a-Remote-Source-to-a-Project) to your project.
2. [Configure a remote field](#Step-2-Configure-a-Remote-Field-in-a-Model) in a model.
3. [Fetch a remote field](#Step-3-Retrieve-a-Remote-Field) using a query.

<p align="center">
<img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/Configure-Remote-Fields-Workflow.png" alt="Workflow for configuring remote fields">
</p>
  
### Step 1. Add a Remote Source to a Project

To add a remote source to a project:
1. In GraphCMS, navigate to the **Schema Builder**.
2. On the left panel under **Schema**, click **Add** next to **Remote Sources**.
3. Enter the following details for a remote source:
   * **Display name** to be shown in GraphCMS.
   * **Prefix** that will be used in the names of custom types in the remote source. A prefix is generated automatically.
   * _(Optional)_ **Description** visible to content editors and API users.
4. _(Optional)_ Enable **debugging** for the remote source to get more information in case of errors. Disable this option once the remote source is properly configured.
5. Select the **API type** the remote source will connect to: **REST** or **GraphQL**.
5. Enter a **base URL** to use in all fields for the remote source.
6. _(Optional)_ Add HTTP **headers** that will be added to all API requests for all remote fields connected to this remote source. Authorization details and accepted media types are common examples of headers.
7. Specify other parameters depending on the API type:
   * **REST**: Under **Custom Type Definitions**, add custom types to enable mapping from the API responses to the GraphQL schema. You can also define customer input types here (optional).
   * **GraphQL**: _(Optional)_ Provide an alternative introspection URL and custom headers to send to the introspection endpoint. If no additional introspection URL is entered, the base URL will be used.<br/><br/>
      > In most GraphQL APIs, querying and introspection on the same URL is allowed by default.<br/>
8. Once finished, click **Create** in the upper-right corner.

> **Example**
> <details><summary><b>Click to view an example configuration of a remote source.</b></summary>
> 
> <img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/Create-a-Remote-Source.png">
> </details>

### Step 2. Configure a Remote Field in a Model

Now you are ready to add a remote field to a model. You can configure remote fields only after you created at least one [remote source](#Step-1-Add-a-Remote-Source-to-a-Project) of the corresponding type — REST or GraphQL.

Follow these instructions depending on the selected API type:
* [REST](#Rest)
* [GraphQL](#GraphQL)
  
#### REST

To configure a remote field for a REST remote source:
1. [How to navigate to this section.]
2. Select a model to which you want to add a remote field, and then click **REST** on the right panel.
3. In the **Create Field** overlay, enter the following details for a field:
   * **Display name** visible to [more information is needed.]
   * **API ID**. [More information is needed.]
   * _(Optional)_ **Description**.
4. Configure the following options:
   * **Make field required**: Triggers an error for a query to GraphCMS if the remote API returns an unsuccessful response. Enable this option if the remote field value is critical for the proper functioning of the frontend application.
   * **Allow multiple values**: Allow the remote API to return an array of values for the selected custom type instead of a single object.
5. Select a **remote source** from which data will be fetched.
6. Select an HTTP **method**.
7. In the **Return Type** field, select one of the **custom types** that you configured for a remote source. This custom type needs to (partially) match with the response of the API path that will be requested in this field. Alternatively, you can set the response to be a scalar type, such as string, boolean, or JSON.
8. _(Optional)_ Add **input arguments**. 
   * Specify the `apiId` for input arguments, and select a previously configured custom input type for the remote source. The `apiId` can be used in the configuration of the URL path.
   * You can add multiple input arguments. See an [example](#Example-Configuration-Using-Custom-Input-Types) for more details.
9. Enter a **path** to be queried for the remote field. This path is added to the remote source base path to form a resulting endpoint.
   * In the path definition, you can use handlebars notation to use fields from the document or from the input arguments if defined. Start by typing a left curly bracket `{`. This is how you can dynamically build a URL path using field values from the same content model or from an input parameter value. As an example, if the model has a field called `userId`, you can build a path that looks as follows: `/users/{{doc.userId}}/repos`.<br/>
> **Steps for an advanced configuration**</br>
> Follow the steps below to complete an <b>advanced configuration</b> of a REST remote source. [More information is needed for each of these parameters.]
10. Add **additional HTTP headers**. You can configure HTTP headers on a remote source or add additional headers to a specific remote field.
    * The header value on the remote field takes precedence over the value on the remote source — if both values are set.
    * You can send all client headers in GraphCMS to the remote source. For example, this can be useful to send user context to the remote server.
11. Configure **cache control** settings. By default, GraphCMS caches queries that include remote fields using a TTL cache with a value of 15 minutes. You can overwrite the TTL in the remote field settings overlay.
    * The minimum TTL value is 60 seconds.
    * A cache control response header sent from a remote source overrides the cache configuration in GraphCMS.
12. Set the **field visibility**: [More information is needed.]
    * `read only` (default): The remote field appears in the content form with a link to the API Playground.
    * `api only`: The remote field does not appear in the content form. You can query it via the API.
13. When ready, click **Create**.
  
> **Example**
> <details><summary><b>Click to view an example configuration of a REST remote field.</b></summary>
>
> <img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/Configure-a-REST-Remote-Field.png" alt="Configuring a REST remote field">
> </details>
  
Go to the next step to find out [how to query a remote field](#Step-3-Retrieve-a-Remote-Field).

#### GraphQL
  
To configure a remote field for a QraphQL remote source:
1. [How to navigate to this section.]
2. Select a model to which you want to add a remote field, and then click **GraphQL** on the right panel.
3. In the **Create Field** overlay, enter the following details for a field:
   * **Display name** visible to [more information is needed.]
   * **API ID**. [More information is needed.]
   * _(Optional)_ **Description**.
4. Choose whether you want to enable the following option:
   * **Make field required**: Triggers an error for a query if the remote API returns a null value. Enable this option if the remote field value is critical for the proper functioning of the front-end application. In this case, the query to GraphCMS will return an error if the remote field does not provide a successful response.
5. Select a **remote source** from which data will be fetched.
6. Pick an HTTP **method**.
7. _(Optional)_ Add **input arguments**. 
    * Specify the `apiId` for input arguments, and select a previously configured custom input type for the remote source. The `apiId` can be used in the configuration of the URL path.
    * You can add multiple input arguments. See an [example](#Example-Configuration-Using-Custom-Input-Types) for more details.  
8. Under **Entry Point**, select a query that will be the entry point into the remote schema. The query tree populates using introspection and shows all available queries in the remote source.
    * When selecting a query, the tree expands to show all ***arguments*** for that query (in purple),  available **sub-queries** (enabled and shown in blue), and available **fields** or scalars (disabled and shown in grey). All fields can be queried through the remote field in GraphCMS.<br/>
        <p align="center"><img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/Arguments-in-a-GraphQL-Remote-Field.png" alt="Configuring arguments"></p>
    * Required arguments are shown with a purple asterisk * next to their ID — even though there is no value validation in GraphCMS. You can use handlebars notation in a parameter field. Autocompletion is currently not supported, but we plan to add it soon.<br/>
    * For queries that return a single value, you can select a sub-query as the entry point. In this case, only fields inside the selected sub-query can be queried through GraphCMS.<br/>
> **Steps for an advanced configuration**</br>
> Follow the steps below to complete an <b>advanced configuration</b> of a REST remote source. [More information is needed for each of these parameters.]<br/>
9. Add **additional HTTP headers**. You can configure HTTP headers on a remote source or add additional headers to a specific remote field.
     * The header value on the remote field takes precedence over the value on the remote source — if both values are set.
     * You can forward all client headers in GraphCMS to the remote source. For example, this can be useful to send user context to the remote server.
10. Configure **cash control** settings. By default, GraphCMS caches queries that include remote fields using a TTL cache with a value of 15 minutes. You can overwrite the TTL in the remote field settings overlay.
     * The minimum TTL value is 60 seconds.
     * A cache control response header sent from a remote source overrides the cache configuration in GraphCMS.
11. Set the **field visibility**:
     * `read only` (default): The remote field appears in the content form with a link to the API playground.
     * `api only`: The remote field does not appear in the content form. You can query it via the API.
12. When ready, click **Create**.
  
> **Example**
> <details><summary><b>Click to view an example configuration of a GraphQL remote field.</b></summary>
>
> <img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/Configure-a-GraphQL-Remote-Field.png">
> </details>

### Step 3. Retrieve a Remote Field
  
Once the remote field is configured, it is added to the GraphCMS schema. You can retrieve the field via the API using a query.
  
```GraphQL
  query MyQuery {
    author(where: {id: "ckadqdbhk00go0148zzxh4bbq"}) {
      id
      name
      githubUserId
      githubUserDetails(githubInput: {userId: "graphcms"}) {
        name
        bio
      }
    }
  }
```

To view available sub-fields inside the remote field, press CTRL+Space or open the **Explorer** view. Note that the remote source prefix is added before the type for easy identification.

In the image below, you can see an example query with remote fields for the Github API.
  
<p align="center">
<img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/View-Available-Subfields.png" alt="Example query">
</p>

## Example Configuration: Using Custom Input Types

In this example, you can see how to define a custom input type on a REST remote field. We will use the following endpoint from the Github REST API:
  
[GET /user](https://docs.github.com/en/rest/users/users#list-users)

1. Configure GitHub as a **remote source** for your project.<br/>
   a. [Create a new remote source](#Step-1-Add-a-Remote-Source-to-a-Project) using [https://api.github.com](https://api.github.com) as the **base URL**.<br/>
   b. Add a **custom type** with the following SDL:<br/>
      <details><summary><b>Click to view the SDL.</b></summary>
      
      ```GraphQL
      type User {
        avatar_url: String
        bio: String
        blog: String
        company: String
        created_at: DateTime
        email: String
        events_url: String
        followers: Int
        followers_url: String
        following: Int
        following_url: String
        gists_url: String
        gravatar_id: String
        hireable: Boolean
        html_url: String
        id: Int
        location: String
        login: String
        name: String
        node_id: String
        organizations_url: String
        public_gists: Int
        public_repos: Int
        received_events_url: String
        repos_url: String
        site_admin: Boolean
        starred_url: String
        subscriptions_url: String
        twitter_username: String
        type: String
        updated_at: DateTime
        url: String
      }
      ```
      </details>
   c. Add a <b>custom input type</b> with the following SDL:<br/>

    ```GraphQL
    input githubInput {
      userId: String!
    }
    ```
2. [Configure a REST remote field](#Rest) in the corresponding model. Below are values we use in the example model:<br/>
   * Model: `Author`.
   * `apiId` for the remote field: `githubUserDetails`.
   * Input type of the input argument: `githubInput`
   * Auto-generated API ID for the input argument: `githubInput`.
   * The field is set as required because we want the path to depend on the input value.<br/>
      <p align="center">
      <img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/Configure-Input-Arguments.png" alt="Configuring remote field parameters">
      </p>
   * The remote field path is configured to use the input argument. You can use the autocomplete feature in the handlebars notation for this.<br/>
      <p align="center">
      <img src="https://github.com/anastasiya-dashuk/GraphCMS-Challenge/blob/main/assets/Configure-Path.png" alt="Configuring a remote field path">
      </p>
3. Make sure there is at least one entry for the selected model, and then **query the remote field** in the API Playground. Use the input argument in a query as shown below.
    ```GraphQL
    query MyQuery {
      author(where: {id: "<some_id>"}) {
        id
        githubUserDetails(githubInput: {userId: "graphcms"}) {
          name
          bio
        }
      }
    }
    ```

    > Verify that you enter a valid `userId`. This parameter is set as required. You can identify this by locating the `!` symbol in the `userId: String!` row. An invalid or null value results in an error.
