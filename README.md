# useQueryParam

import { useCallback, useEffect, useState } from "react";
import useRouter, { DEFAULT_OPTION } from "hooks/useRouter";
import _ from "lodash";
import queryString from "query-string";

export const isUrlValueValid = (value) => {
    if (_.isArray(value) || _.isObject(value)) return !_.isEmpty(value);
    return !_.isNil(value);
};

const getLocationSearch = () => {
    return window.location.search;
}

const getQueryParams = () => {
    return queryString.parse(getLocationSearch(), DEFAULT_OPTION);
}

const getQueryParamValue = (key, parser) => {
    return getValueParsed(getQueryParams()[key], parser)
}

const getValueParsed = (value, parser) => {
    if (parser) value = parser(value);
    return value
}

// need to use window.location.search because with the useLocation() there is a problem if we use many times useQueryParam (problem of sync)
export default function useQueryParam(key, defaultValue, parser, serializer) {
    const router = useRouter();

    const [value, setValue] = useState(() => {
        return getQueryParamValue(key, parser) || /*localstorage*/ defaultValue
    });

    // constructor
    useEffect(() => {
        if (!getQueryParamValue(key) && !_.isNil(defaultValue)) {
            onSetValue(defaultValue);
        }
    }, []);

    // when url change
    useEffect(() => {
        const queryParamValue = getQueryParamValue(key);
        if (queryParamValue) {
            const valueParsed = getValueParsed(queryParamValue, parser);
            if (!_.isEqual(valueParsed, value)) setValue(valueParsed);
        }
    }, [getLocationSearch()]);

    useEffect(() => {
        //use localStorage
    }, [value]);

    const onSetValue = useCallback(
        (newValue, push = false) => {

            const queryParamValue = getQueryParamValue(key, parser)

            if (_.isEqual(newValue, queryParamValue)) return;

            if (serializer) newValue = serializer(newValue);

            if (isUrlValueValid(newValue)) {
                const routerParams = {
                    pathname: router.pathname,
                    search: router.getSearchString({
                        ...getQueryParams(),
                        [key]: newValue
                    })
                };
                if (push) router.push(routerParams);
                else router.replace(routerParams);
            } else {
                setValue(newValue);
            }
        },
        []
    );

    return [value, onSetValue];
}
