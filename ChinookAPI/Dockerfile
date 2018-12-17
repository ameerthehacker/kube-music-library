FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80

FROM microsoft/dotnet:2.1-sdk AS build
WORKDIR /src
COPY ChinookAPI/ChinookAPI.csproj ChinookAPI/
RUN dotnet restore ChinookAPI/ChinookAPI.csproj
COPY . .
WORKDIR /src/ChinookAPI
RUN dotnet build ChinookAPI.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish ChinookAPI.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "ChinookAPI.dll"]
