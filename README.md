So now we can write webservices with Clarive rulebooks. We call them _webhooks_, but they are actually __inbound__.

Let me explain. Let's start with the problem at hand.

### The Problem at hand

With Clarive you can write your automation as rulebook files. (You can also write them visually with Clarive EE, but that's another story.) Rulebooks can do wonderful things, like automating your pipeline: building, testing and deploying apps. They can also be used to __provision__ your infrastructure (think cloud instance or DB) and can be triggered by many different rules, like a topic modification or a branch being pushed. They can also be scheduled to run on a cron.

But what if you want to trigger them from an outside event? Say when a user opens an issue on your issue tracker, or someone pushes something to a repository somewhere? That's when we wanted to trigger a rule in our rulebook. To do that you can call into Clarive. But instead of running a generic Clarive API that calls a rulebook, we wanted to expose __meaningful urls__ that would make more sense of 

### What is a Rulebook

Rulebook is a file (or set of files) checked into a Git repository in your Clarive instance. It all starts with the `.clarive.yml` file.

Your rulebook will contain _rules_. Rules can be `build`, `test`, `deploy`, or events like `topic_modify` or `repository_update`. And then they can be webhook rules. Webhook rules are any events that start with a `/` slash character. The event name will become an __url fragment__ to be called into.

Webhook rules behave like exposed webservices right into your instance, running within your repository.

Here's an example:

```yaml
/hello_world:
    - echo: running the hello world webhook
   
    # do some meaningful stuff here

    - web_response:
         # return something to the caller
           
```

### Anatomy of a web call

When you call into a rule webhook introducing a url like this:
<code>      https://{myclariveserver}/rule/json/{MyProject}/{MyRepo}/hello_world?api_key={my_user_api_key} </code>

It goes through the following steps:

- Authenticates the user (users must be authenticated - via api_key).

<img src="https://clarive.com/wp-content/uploads/api_key.png" alt="your API key" width="1143" height="334" class="size-full wp-image-13744" /> This is your user API key

- Locate your project `MyProject` and the repository associated `MyRepo`
- Find `.clarive.yml` file
- Look for a webservice defined with `/` and called `hello_world`
- Execute the `echo` code and return the result in `web_response`

### A hardcore example: provision an Azure VM

To run this example you can create an instance in our free cloud. That will give you the complete infrastructure you need.

1. [Get a free Azure account](https://azure.microsoft.com/en-us/free/) if you don't have one already.
     - [Create a service principal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal) that allows Clarive to authenticate in your Azure.
     - [Create a resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-portal) needed to create your virtual machine.
2. [Get your free Clarive cloud instance](/free-cloud)
3. Once you get an email with the instructions, you can login in your clarive.
4. Create a new project and a git repository associated to it.
5. Create a story from your project, and that will create automatically a new branch in your repo

    <img src="https://clarive.com/wp-content/uploads/topic.png" alt="your topic" width="620" height="401" class="size-full wp-image-13735" /> Create a topic in Clarive

6. Create your azure variables that you will need to login using a service principal

    <img src="https://clarive.com/wp-content/uploads/vars.png" alt="Azure vars" width="1455" height="284" class="size-full wp-image-13734" /> Your secret vars that allow to login in your Azure
7. Clone this repo in your local workspace, checkout the actual branch and modify your <em>.clarive.yml</em> with the webservice.

    ```yaml
/create_azure_vm:
    image: microsoft/azure-cli #docker image to run az commands
    do:
        - myoutput = shell: |
              az login --service-principal -u {{ ctx.var('az-service-principal') }}  --password {{ ctx.var('az-password') }}  --tenant {{ ctx.var('az-tenant') }}
              az vm create --resource-group <your_resource_group> --name myVM --image UbuntuLTS --generate-ssh-keys
        - if "{{ myoutput.rc == 0 }}"
          then: 
            - result =: "VM created succesfully!"
          else:
            - result =:"Upps! something went wrong, review your azure instance."
        - web_response:
            body: ${result} 
   ```
8. Push the changes (after commiting them, obviously) into your brach, then you have to take your __topic to PROD environment__ where your branch will be __merged in master__ or do it manually.

9. Your defined webservice will be available so call it with your browser or in your command line using "curl".

   ```console
curl https://<your_cloud_instance>.clarive.io/rule/json/<your_project>/<your_repo>/create_azure_vm?api_key=<your_user_api_key>
   ```

10. Finally, you can see your VM created in your Azure instance.

   [caption id="attachment_13754" align="alignnone" width="799"]<img src="https://clarive.com/wp-content/uploads/azure_vm_created.png" alt="" width="875" height="399" class="alignnone size-full wp-image-13754" />Your VM created in Azure with a Clarive webhook[/caption]

That's it. You've just created your first DevOps webservice in Clarive :)

<hr />
<note>Get your free Clarive instance and create your own DevOps webhooks</note>
<hr />
