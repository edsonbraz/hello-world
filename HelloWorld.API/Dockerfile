# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081


# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
ARG BUILD_CONFIGURATION=Release
ARG GITHUB_TOKEN
ARG GITHUB_ACTOR
ARG GITHUB_REPOSITORY_OWNER

WORKDIR /src
COPY ["HelloWorld.API/HelloWorld.API.csproj", "HelloWorld.API/"]

# Add GitHub NuGet source if GitHub token is provided
RUN if [ -n "$GITHUB_TOKEN" ] && [ -n "$GITHUB_ACTOR" ] && [ -n "$GITHUB_REPOSITORY_OWNER" ]; then \
    dotnet nuget add source --username $GITHUB_ACTOR --password $GITHUB_TOKEN --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$GITHUB_REPOSITORY_OWNER/index.json"; \
    fi

RUN dotnet restore "./HelloWorld.API/HelloWorld.API.csproj"
COPY . .
WORKDIR "/src/HelloWorld.API"
RUN dotnet build "./HelloWorld.API.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./HelloWorld.API.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "HelloWorld.API.dll"]