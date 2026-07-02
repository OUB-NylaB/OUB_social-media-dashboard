/**
 * OneUnited Suite — API Proxy (Cloudflare Worker)
 * ------------------------------------------------
 * This is the "backend relay" the dashboard needs for platforms that block
 * direct browser calls (Meta, Instagram, TikTok, LinkedIn, GA4, Sprout).
 *
 * How it works:
 *  - Your API tokens live here, as encrypted Worker secrets — NEVER in the
 *    browser, never in your dashboard's code, never visible to site visitors.
 *  - The dashboard calls THIS worker's URL. This worker adds the secret
 *    token and forwards the request to the real platform, then hands the
 *    response back with CORS headers the browser will accept.
 *
 * IMPORTANT — the exact paths below are starting templates, not finished
 * integrations. Each platform's real API has its own required parameters,
 * report formats, and permissions (especially GA4 and Sprout, which need a
 * specific report/query shape). Test each route against that platform's own
 * API docs and adjust the `target` URL / body as needed before relying on it.
 */

const ALLOWED_ORIGIN = "*"; // 🔒 Once deployed, replace * with your actual
                             // GitHub Pages origin, e.g. "https://oub-nylab.github.io"

function corsHeaders() {
  return {
    "Access-Control-Allow-Origin": ALLOWED_ORIGIN,
    "Access-Control-Allow-Methods": "GET,POST,OPTIONS",
    "Access-Control-Allow-Headers": "Content-Type",
  };
}

async function relay(target, init, cors) {
  const resp = await fetch(target, init);
  const body = await resp.text();
  return new Response(body, {
    status: resp.status,
    headers: { ...cors, "Content-Type": "application/json" },
  });
}

export default {
  async fetch(request, env) {
    const cors = corsHeaders();
    const url = new URL(request.url);

    if (request.method === "OPTIONS") {
      return new Response(null, { headers: cors });
    }

    try {
      switch (url.pathname) {
        // ---- META (Facebook Pages) -------------------------------------
        // Usage from dashboard: GET {WORKER_URL}/meta?path=/me/accounts
        case "/meta": {
          const path = url.searchParams.get("path") || "/me";
          const sep = path.includes("?") ? "&" : "?";
          const target = `https://graph.facebook.com/v19.0${path}${sep}access_token=${env.META_TOKEN}`;
          return await relay(target, {}, cors);
        }

        // ---- INSTAGRAM (via Graph API, linked Facebook Page) -----------
        case "/instagram": {
          const path = url.searchParams.get("path") || "/me";
          const sep = path.includes("?") ? "&" : "?";
          const target = `https://graph.facebook.com/v19.0${path}${sep}access_token=${env.INSTAGRAM_TOKEN}`;
          return await relay(target, {}, cors);
        }

        // ---- TIKTOK (Business/Content API) -----------------------------
        case "/tiktok": {
          const path = url.searchParams.get("path") || "/";
          const target = `https://open.tiktokapis.com/v2${path}`;
          return await relay(
            target,
            { headers: { Authorization: `Bearer ${env.TIKTOK_TOKEN}` } },
            cors
          );
        }

        // ---- LINKEDIN (Marketing API) -----------------------------------
        case "/linkedin": {
          const path = url.searchParams.get("path") || "/me";
          const target = `https://api.linkedin.com/v2${path}`;
          return await relay(
            target,
            { headers: { Authorization: `Bearer ${env.LINKEDIN_TOKEN}` } },
            cors
          );
        }

        // ---- GA4 (Data API — POST report request) -----------------------
        // Usage: POST {WORKER_URL}/ga4  with your report JSON body
        case "/ga4": {
          const target = `https://analyticsdata.googleapis.com/v1beta/properties/${env.GA4_PROPERTY_ID}:runReport`;
          const body = await request.text();
          return await relay(
            target,
            {
              method: "POST",
              headers: {
                Authorization: `Bearer ${env.GA4_TOKEN}`,
                "Content-Type": "application/json",
              },
              body,
            },
            cors
          );
        }

        // ---- SPROUT SOCIAL ------------------------------------------------
        case "/sprout": {
          const path = url.searchParams.get("path") || "/";
          const target = `https://api.sproutsocial.com${path}`;
          return await relay(
            target,
            { headers: { Authorization: `Bearer ${env.SPROUT_TOKEN}` } },
            cors
          );
        }

        default:
          return new Response(
            JSON.stringify({ error: "Unknown route. Try /meta, /instagram, /tiktok, /linkedin, /ga4, /sprout" }),
            { status: 404, headers: { ...cors, "Content-Type": "application/json" } }
          );
      }
    } catch (err) {
      return new Response(
        JSON.stringify({ error: err.message }),
        { status: 500, headers: { ...cors, "Content-Type": "application/json" } }
      );
    }
  },
};
