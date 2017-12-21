To retrieve the backlog for a team

project/team/_api/_backlog/payload?__v=5&level=Stories&includeParents=false

Retrieve the following information from the JSON:
* queryResults.wiql            : the WI query used to retrieve the data
* queryResults.pageColumns     : the columns displayed on the page
* queryResults.payload.columns : the columns returned by the query
* queryResults.payload.rows    : the rows returned by the query
* queryResults.linkIds         : the type of each link
* queryResults.ownedIds        : the list of work item IDs that are owned by the current team
* queryResults.sourceIds       : the list of parent work items in the same order as the target IDs and link IDs
* queryResults.targetIds       : the list of child work items in the same order as the source IDs and link IDs


To get users with a Test Manager licence

_apiusermanagement/GetAccountExtensionUsers?extensionId=ms.vss-testmanager-web&mkt=en-us

Returns a JSON list of users




To add users to the Test Manager licence

_apiusermanagement/AddMultipleUsersToExtension

foreach(var u in usersToAdd)
{
    users.Add(new SerializableUser
    {
        DisplayName = u.DisplayName,
        ObjectId = u.Id,
        UserName = u.Properties["account"].ToString().ToUpperInvariant()
    });
}

var data = new Dictionary<string, string>
{
    ["serializedUsers"] = usersJson,
    ["extensionId"] = "ms.vss-testmanager-web",
    ["__RequestVerificationToken"] = context.RequestValidationToken
};



To remove users to the Test Manager licence

_apiusermanagement/RemoveUserFromExtension

var data = new List<KeyValuePair<string, string>>();
foreach (var user in usersToRemove)
{
    data.Add(new KeyValuePair<string, string>("userIds", user.Id.ToString()));
}
data.Add(new KeyValuePair<string, string>("extensionId", "ms.vss-testmanager-web"));
data.Add(new KeyValuePair<string, string>("__RequestVerificationToken", context.RequestValidationToken));



To get the security data necessary for the private API

using (var handler = new HttpClientHandler { UseCookies = false })
using (var client = new HttpClient(handler) { BaseAddress = new Uri(collectionOption.Value()) })
{
    var url = "_admin/_userHub";
    var personalAccessToken = patOption.Value();

    var base64Encoding = Convert.ToBase64String(ASCIIEncoding.ASCII.GetBytes(string.Format("{0}:{1}", "", personalAccessToken)));
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", base64Encoding);

    using (HttpResponseMessage response = client.GetAsync(url).Result)
    {
        response.EnsureSuccessStatusCode();
        var responseBody = await response.Content.ReadAsStringAsync();

        context.Cookies = response.Headers.GetValues("Set-Cookie");

        // Look for __RequestVerificationToken element
        var document = new HtmlDocument();
        document.LoadHtml(responseBody);

        context.RequestValidationToken = document.DocumentNode.Descendants("input")
                                                              .Where(i => i.Attributes.AttributesWithName("name").Any() &&
                                                                          i.Attributes.AttributesWithName("value").Any() &&
                                                                          i.Attributes["name"].Value == "__RequestVerificationToken")
                                                              .Select(i => i.Attributes["value"].Value)
                                                              .FirstOrDefault();
    }
}
