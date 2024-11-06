# Stage 1: Build the application
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /app

COPY ./*.csproj ./
RUN dotnet restore

COPY . ./

RUN dotnet publish -c Release -o out

# Stage 2: Run the application
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2-stretch-slim
WORKDIR /app
COPY --from=build /app/out ./

ENTRYPOINT ["dotnet", "aspnet-core-dotnet-core.dll"]

