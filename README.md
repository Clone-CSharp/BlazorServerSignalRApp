# BlazorServerSignalRApp
Clone  https://learn.microsoft.com/ko-kr/aspnet/core/blazor/tutorials/signalr-blazor?view=aspnetcore-7.0&amp;tabs=visual-studio-code&amp;pivots=server

이 자습서에서는 Blazor와 함께 SignalR을 이용해서 실시간 앱을 구현하기 위한 기본 사항을 알려 줍니다.

방법 배우기:
> Blazor 프로젝트 만들기
> SignalR 클라이언트 라이브러리 추가
> SignalR 허브 추가
> SignalR 서비스 및 SignalR 허브에 대한 엔드포인트 추가
> 채팅을 위한 Razor 구성 요소 코드 추가

1. Blazor Server 앱 만들기
   - dotnet new blazorserver -o BlazorServerSignalRApp
  
2. SignalR 클라이언트 라이브러리 추가
   - dotnet add package Microsoft.AspNetCore.SignalR.Client
   
3. SignalR 허브 추가
  - Hubs/ChatHub.cs 생성

  using Microsoft.AspNetCore.SignalR;

  namespace BlazorServerSignalRApp.Server.Hubs
  {
      public class ChatHub : Hub
      {
          public async Task SendMessage(string user, string message)
          {
              await Clients.All.SendAsync("ReceiveMessage", user, message);
          }
      }
  }
  
4. 서비스 및 SignalR 허브에 대한 엔드포인트 추가
    - Program.cs
    
    - using Microsoft.AspNetCore.ResponseCompression;
      using BlazorServerSignalRApp.Server.Hubs;
  
    - builder.Services.AddRazorPages();
      builder.Services.AddServerSideBlazor();
      builder.Services.AddSingleton<WeatherForecastService>();
      builder.Services.AddResponseCompression(opts =>
      {
        opts.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
            new[] { "application/octet-stream" });
      });

   -  builder.Services.AddRazorPages();
      builder.Services.AddServerSideBlazor();
      builder.Services.AddSingleton<WeatherForecastService>();
      builder.Services.AddResponseCompression(opts =>
      {
          opts.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
              new[] { "application/octet-stream" });
      });
      
    - app.UseResponseCompression();

      if (!app.Environment.IsDevelopment())
      {
          app.UseExceptionHandler("/Error");
          app.UseHsts();
      }

      app.UseHttpsRedirection();

      app.UseStaticFiles();

      app.UseRouting();

      app.MapBlazorHub();
      app.MapHub<ChatHub>("/chathub");
      app.MapFallbackToPage("/_Host");

      app.Run();      
      
      
      
      - Pages/Index.razor
      
       - @page "/"
         @using Microsoft.AspNetCore.SignalR.Client
         @inject NavigationManager Navigation
         @implements IAsyncDisposable

         <PageTitle>Index</PageTitle>

         <div class="form-group">
             <label>
                 User:
                 <input @bind="userInput" />
             </label>
         </div>
         <div class="form-group">
             <label>
                 Message:
                 <input @bind="messageInput" size="50" />
             </label>
         </div>
         <button @onclick="Send" disabled="@(!IsConnected)">Send</button>

         <hr>

         <ul id="messagesList">
             @foreach (var message in messages)
             {
                 <li>@message</li>
             }
         </ul>

         @code {
             private HubConnection? hubConnection;
             private List<string> messages = new List<string>();
             private string? userInput;
             private string? messageInput;

             protected override async Task OnInitializedAsync()
             {
                 hubConnection = new HubConnectionBuilder()
                     .WithUrl(Navigation.ToAbsoluteUri("/chathub"))
                     .Build();

                 hubConnection.On<string, string>("ReceiveMessage", (user, message) =>
                 {
                     var encodedMsg = $"{user}: {message}";
                     messages.Add(encodedMsg);
                     InvokeAsync(StateHasChanged);
                 });

                 await hubConnection.StartAsync();
             }

             private async Task Send()
             {
                 if (hubConnection is not null)
                     {
                         await hubConnection.SendAsync("SendMessage", userInput, messageInput);
                     }
             }

             public bool IsConnected =>
                 hubConnection?.State == HubConnectionState.Connected;

             public async ValueTask DisposeAsync()
             {
                 if (hubConnection is not null)
                 {
                     await hubConnection.DisposeAsync();
                 }
             }
         }      
