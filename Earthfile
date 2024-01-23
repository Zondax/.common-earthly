#
VERSION 0.8
FROM python:3.10-alpine3.18

RUN apk add git

# This shared command can be used to publish images in a standardized format
# this will publish images named as zondax/${CONTAINER_FULLNAME}-{FLEXTAGS}
PUBLISH_WITH_FLEXTAGS:
    COMMAND
    ARG --required CONTAINER_FULLNAME
    ARG --required EARTHLY_GIT_SHORT_HASH
    ARG EARTHLY_GIT_COMMIT_TIMESTAMP
    ARG EARTHLY_GIT_TAG
    ARG EARTHLY_GIT_BRANCH

    # Log all the tags before removing duplicates
    RUN echo \
        "\n\n\n" \
        "------------------------------------\n" \
        "CONTAINER_FULLNAME          : $CONTAINER_FULLNAME \n" \
        "EARTHLY_GIT_SHORT_HASH      : $EARTHLY_GIT_SHORT_HASH \n" \
        "EARTHLY_GIT_COMMIT_TIMESTAMP: $EARTHLY_GIT_COMMIT_TIMESTAMP \n" \
        "EARTHLY_GIT_TAG             : $EARTHLY_GIT_TAG \n" \
        "EARTHLY_GIT_BRANCH          : $EARTHLY_GIT_BRANCH \n" \
        "------------------------------------\n" \
        "\n\n\n"

    # This will detect there is already a tag (:) and will use a dash - instead
    ENV DELIMITER=':'
    IF echo "$CONTAINER_FULLNAME" | grep -q ":"
        ENV DELIMITER='-'
    END

    # Generate tags
    ENV GIT_TAG=""
    ENV TIMESTAMP_TAG=""
    ENV SHORT_HASH_TAG=""
    ENV BRANCH_TAG=""
    ENV NO_TAG=""

    IF [ ! -z "$EARTHLY_GIT_TAG" ]
        ENV GIT_TAG="zondax/${CONTAINER_FULLNAME}${DELIMITER}${EARTHLY_GIT_TAG}"
    END

    IF [ ! -z "$EARTHLY_GIT_COMMIT_TIMESTAMP" ]
        ARG TIMESTAMP=$(date -d @${EARTHLY_GIT_COMMIT_TIMESTAMP} +"%Y%m%d%H%M%S")
        ENV TIMESTAMP_TAG="zondax/${CONTAINER_FULLNAME}${DELIMITER}T${TIMESTAMP}"
    END

    ENV SHORT_HASH_TAG="zondax/${CONTAINER_FULLNAME}${DELIMITER}${EARTHLY_GIT_SHORT_HASH}"

    ARG TMP_BRANCH=$(echo "$EARTHLY_GIT_BRANCH" | tr '/' '-')
    ENV BRANCH_TAG="zondax/${CONTAINER_FULLNAME}${DELIMITER}${TMP_BRANCH}"

    IF [ "$EARTHLY_GIT_BRANCH" = "main" ]
        ENV NO_TAG="zondax/${CONTAINER_FULLNAME}"
    END

    # Log all the tags after removing duplicates
    RUN echo \
        "\n\n\n" \
        "------------------------------------\n" \
        "GIT_TAG       : $GIT_TAG \n" \
        "TIMESTAMP_TAG : $TIMESTAMP_TAG \n" \
        "SHORT_HASH_TAG: $SHORT_HASH_TAG \n" \
        "BRANCH_TAG    : $BRANCH_TAG \n" \
        "NO_TAG        : $NO_TAG \n" \
        "------------------------------------\n" \
        "\n\n\n"

    # Combine all tags into a single variable and remove duplicates
    ARG TAGS=$(echo "${GIT_TAG} ${TIMESTAMP_TAG} ${SHORT_HASH_TAG} ${BRANCH_TAG} ${LATEST_TAG} ${NO_TAG}" | tr ' ' '\n' | sort -u | tr '\n' ' ')

    # Log each deduplicated tag in a different line
    RUN for TAG in $TAGS; do echo $TAG; done

    # Loop through each tag and save the image
    FOR TAG IN $TAGS
        WAIT
            SAVE IMAGE --push $TAG
        END
    END

PUBLISH_NODE:
    COMMAND
    ARG --required BASE_NAME
    ARG --required BUNDLE_VARIANT
    ARG --required NODE_BRANCH_TAG
    ARG NODE_BRANCH_TAG_OVERRIDE

    ENV CONTAINER_NAME=${BASE_NAME}-${BUNDLE_VARIANT}:${NODE_BRANCH_TAG}
    IF [ ! -z "$NODE_BRANCH_TAG_OVERRIDE" ]
        # This will call the UDC and attach standards tags
        ENV CONTAINER_NAME=${BASE_NAME}-${BUNDLE_VARIANT}:${NODE_BRANCH_TAG_OVERRIDE}
    END

    DO +PUBLISH_WITH_FLEXTAGS --CONTAINER_FULLNAME=${BASE_NAME}-${BUNDLE_VARIANT}:${NODE_BRANCH_TAG}
