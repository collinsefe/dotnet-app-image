# Stage 1: Build the application
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /app

# Copy the csproj and restore any dependencies (via NuGet)
# COPY /aspnet-core-dotnet-core/*.csproj ./

COPY ./*.csproj ./
RUN dotnet restore

# Copy the rest of the application code
COPY . ./

# Build the application
RUN dotnet publish -c Release -o out

# Stage 2: Run the application
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2-stretch-slim
WORKDIR /app
COPY --from=build /app/out ./

# Run command
ENTRYPOINT ["dotnet", "aspnet-core-dotnet-core.dll"]


# COPY /aspnet-core-dotnet-core/. ./:
