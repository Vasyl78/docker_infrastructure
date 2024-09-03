FROM ruby:3.2.4-alpine AS build-env

ARG PROJECT_ROOT=/app
ARG BUILD_PACKAGES="build-base curl-dev git"
ARG DEV_PACKAGES="postgresql-dev yaml-dev zlib-dev"
ARG RUBY_PACKAGES="tzdata"

#gcompat
ENV RACK_ENV=production
ENV BUNDLE_APP_CONFIG="$PROJECT_ROOT/.bundle"

WORKDIR $PROJECT_ROOT

RUN apk update \
    && apk upgrade \
    && apk add --update --no-cache $BUILD_PACKAGES $DEV_PACKAGES $RUBY_PACKAGES

RUN ln -s /lib/ld-musl-x86_64.so.1 /lib/ld-linux-x86-64.so.2

RUN gem install -N bundler

COPY Gemfile* ./
COPY Gemfile Gemfile.lock $PROJECT_ROOT/

RUN bundle config --global frozen 1 \
    && bundle install  -j4 --retry 3 --path=vendor/bundle --without development:test:assets \
    # Remove unneeded files (cached *.gem, *.o, *.c)
    && rm -rf vendor/bundle/ruby/3.2.0/cache/*.gem \
    && find vendor/bundle/ruby/3.2.0/gems/ -name "*.c" -delete \
    && find vendor/bundle/ruby/3.2.0/gems/ -name "*.o" -delete

COPY . .

# Remove folders not needed in resulting image
RUN rm -rf tmp spec allure-result

############### Build step done ###############

FROM ruby:3.2.4-alpine

ARG PROJECT_ROOT=/app
ARG PACKAGES="tzdata postgresql-client bash"

ENV RACK_ENV=production
ENV BUNDLE_APP_CONFIG="$PROJECT_ROOT/.bundle"

WORKDIR $PROJECT_ROOT

# install packages
RUN apk update \
    && apk upgrade \
    && apk add --update --no-cache $PACKAGES
COPY --from=build-env $PROJECT_ROOT $PROJECT_ROOT

RUN ln -s /lib/ld-musl-x86_64.so.1 /lib/ld-linux-x86-64.so.2

EXPOSE 9292

CMD ["bundle", "exec", "rackup"]
