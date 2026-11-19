// Cloudflare Pages Function: POST /api/create-payment
// Создаёт платёж в ЮKassa и возвращает ссылку на оплату.
// Ключи берутся из переменных окружения Cloudflare (см. README):
//   YOOKASSA_SHOP_ID, YOOKASSA_SECRET_KEY
// Никогда не храните ключи в этом файле и в репозитории!

export async function onRequest(context) {
  const { request, env } = context;

  if (request.method !== "POST") {
    return Response.json({ error: "Method not allowed" }, { status: 405 });
  }

  const SHOP_ID = env.YOOKASSA_SHOP_ID;
  const SECRET_KEY = env.YOOKASSA_SECRET_KEY;

  if (!SHOP_ID || !SECRET_KEY) {
    return Response.json(
      { error: "Payment is not configured: set YOOKASSA_SHOP_ID and YOOKASSA_SECRET_KEY env vars" },
      { status: 500 }
    );
  }

  let payload = {};
  try { payload = await request.json(); } catch (e) { /* тело не обязательно */ }

  const origin = new URL(request.url).origin;

  const yookassaResponse = await fetch("https://api.yookassa.ru/v3/payments", {
    method: "POST",
    headers: {
      "Authorization": "Basic " + btoa(`${SHOP_ID}:${SECRET_KEY}`),
      "Content-Type": "application/json",
      "Idempotence-Key": crypto.randomUUID(),
    },
    body: JSON.stringify({
      amount: { value: "55.00", currency: "RUB" },
      capture: true,
      confirmation: {
        type: "redirect",
        return_url: origin + "/?payment=success",
      },
      description: payload.description || "Поддержка Spy Save (премиум 1 мес)",
      metadata: payload.metadata || {},
    }),
  });

  const data = await yookassaResponse.json();

  if (!yookassaResponse.ok || !data.confirmation || !data.confirmation.confirmation_url) {
    return Response.json(
      { error: "YooKassa error", detail: data },
      { status: 502 }
    );
  }

  return Response.json({ confirmation_url: data.confirmation.confirmation_url });
}
