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
  
4. 
