FROM mcr.microsoft.com/azure-functions/dotnet:4-appservice AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
ENV USER=huston
ENV GID=1000
ENV UID=2000
ENV COMPlus_EnableDiagnostics=0
ADD helloworkflow /app/publish/helloworkflow
COPY ["src.csproj", "workflow/"]
RUN dotnet restore "workflow/src.csproj"
COPY . .
WORKDIR "/src/workflow"
RUN dotnet build "src.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "src.csproj" -c Release -o /app/publish
RUN addgroup --group texas --gid ${GID} \
&& adduser \    
    --uid "${UID}" \
    --gid "${GID}" \
    "${USER}" 
RUN chown huston:texas /app
USER huston:texas

ENTRYPOINT ["dotnet", "src.dll"]

FROM base AS final
WORKDIR /home/site/wwwroot
COPY --from=publish /app/publish/ .
ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
    AzureFunctionsJobHost__Logging__Console__IsEnabled=true 
